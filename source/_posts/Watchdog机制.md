---
title: Watchdog机制
date: 2018-07-15 16:06:29
categories: Android Framework
tags: [Android]
toc: true
description: The great pleasure in life is doing what people say you cannot do. 
---
Watchdog的中文的“看门狗”，有保护的意思。最早引入Watchdog是在单片机系统中，由于单片机的工作环境容易受到外界磁场的干扰，导致程序“跑飞”，造成整个系统无法正常工作，因此，引入了一个“看门狗”，对单片机的运行状态进行实时监测，针对运行故障做一些保护处理，譬如让系统重启。这种Watchdog属于硬件层面，必须有硬件电路的支持
Linux也引入了Watchdog，在Linux内核下，当Watchdog启动后，便设定了一个定时器，如果在超时时间内没有对/dev/Watchdog进行写操作，则会导致系统重启。通过定时器实现的Watchdog属于软件层面。
Android设计了一个软件层面Watchdog，用于保护一些重要的系统服务，当出现故障时，通常会让Android系统重启。由于这种机制的存在，就经常会出现一些system_server进程被Watchdog杀掉而发生手机重启的问题。
### Watchdog初始化
分析Watchdog的实现逻辑。为了描述方便，ActivityManagerService， PackageManagerService， WindowManagerService会分别简称为AMS, PKMS, WMS
代码路径frameworks/base/services/core/java/com/android/server/Watchdog.java，
Android的Watchdog是一个单例线程，在System Server时就会初始化Watchdog。Watchdog在初始化时，会构建很多HandlerChecker，大致可以分为两类：

- Monitor Checker，用于检查是Monitor对象可能发生的死锁, AMS, PKMS, WMS等核心的系统服务都是Monitor对象。
- Looper Checker，用于检查线程的消息队列是否长时间处于工作状态。Watchdog自身的消息队列，Ui, Io, Display这些全局的消息队列都是被检查的对象。此外，一些重要的线程的消息队列，也会加入到Looper Checker中，譬如AMS, PKMS，这些是在对应的对象初始化时加入的。
```
private Watchdog() {
    ....
    mMonitorChecker = new HandlerChecker(FgThread.getHandler(),
                "foreground thread", DEFAULT_TIMEOUT);
    mHandlerCheckers.add(mMonitorChecker);
    mHandlerCheckers.add(new HandlerChecker(new Handler(Looper.getMainLooper()),
                "main thread", DEFAULT_TIMEOUT));
    mHandlerCheckers.add(new HandlerChecker(UiThread.getHandler(),
                "ui thread", DEFAULT_TIMEOUT));
    mHandlerCheckers.add(new HandlerChecker(IoThread.getHandler(),
                "i/o thread", DEFAULT_TIMEOUT));
    mHandlerCheckers.add(new HandlerChecker(DisplayThread.getHandler(),
                "display thread", DEFAULT_TIMEOUT));
    ...
}
```
两类HandlerChecker的侧重点不同，Monitor Checker预警我们不能长时间持有核心系统服务的对象锁，否则会阻塞很多函数的运行; Looper Checker预警我们不能长时间的霸占消息队列，否则其他消息将得不到处理。这两类都会导致系统卡住(System Not Responding)
### 添加WatchDog监测对象
Watchdog初始化以后，就可以作为system_server进程中的一个单独的线程运行了。但这个时候，还不能触发Watchdog的运行，因为AMS, PKMS等系统服务还没有加入到Watchdog的监测集。 所谓监测集，就是需要Watchdog关注的对象，Android中有成千上万的消息队列在同时运行，然而，Watchdog毕竟是系统层面的东西，它只会关注一些核心的系统服务。
Watchdog提供两个方法，分别用于添加Monitor Checker对象和Looper Checker对象:
```
public void addMonitor(Monitor monitor) {
    // 将monitor对象添加到Monitor Checker中，
    // 在Watchdog初始化时，可以看到Monitor Checker本身也是一个HandlerChecker对象
    mMonitors.add(monitor);
}

public void addThread(Handler thread, long timeoutMillis) {
    synchronized (this) {
        if (isAlive()) {
            throw new RuntimeException("Threads can't be added once the Watchdog is running");
        }
        final String name = thread.getLooper().getThread().getName();
        // 为Handler构建一个HandlerChecker对象，其实就是**Looper Checker**
        mHandlerCheckers.add(new HandlerChecker(thread, name, timeoutMillis));
    }
}
```
被Watchdog监测的对象，都需要将自己添加到Watchdog的监测集中。以下是AMS的类定义和构造器的代码片段：
```
public final class ActivityManagerService extends ActivityManagerNative
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {

    public ActivityManagerService(Context systemContext) {
        ...
        Watchdog.getInstance().addMonitor(this);
        Watchdog.getInstance().addThread(mHandler);
    }

    public void monitor() {
        synchronized (this) { }
    }
}
```
AMS实现了Watchdog.Monitor接口，这个接口只有一个方法，就是monitor()，它的作用后文会再解释。这里可以看到在AMS的构造器中，将自己添加到Monitor Checker对象中，然后将自己的handler添加到Looper Checker对象中。 其他重要的系统服务添加到Watchdog的代码逻辑都与AMS差不多。
### Watchdog的监测机制
Watchdog本身是一个线程，它的run()方法实现如下：
```
@Override
public void run() {
    boolean waitedHalf = false;
    while (true) {
        ...
        synchronized (this) {
            ...
            // 1. 调度所有的HandlerChecker
            for (int i=0; i<mHandlerCheckers.size(); i++) {
                HandlerChecker hc = mHandlerCheckers.get(i);
                hc.scheduleCheckLocked();
            }
            ...
            // 2. 开始定期检查
            long start = SystemClock.uptimeMillis();
            while (timeout > 0) {
                ...
                try {
                    wait(timeout);
                } catch (InterruptedException e) {
                    Log.wtf(TAG, e);
                }
                ...
                timeout = CHECK_INTERVAL - (SystemClock.uptimeMillis() - start);
            }

            // 3. 检查HandlerChecker的完成状态
            final int waitState = evaluateCheckerCompletionLocked();
            if (waitState == COMPLETED) {
                ...
                continue;
            } else if (waitState == WAITING) {
                ...
                continue;
            } else if (waitState == WAITED_HALF) {
                ...
                continue;
            }

            // 4. 存在超时的HandlerChecker
            blockedCheckers = getBlockedCheckersLocked();
            subject = describeCheckersLocked(blockedCheckers);
            allowRestart = mAllowRestart;
        }
        ...
        // 5. 保存日志，判断是否需要杀掉系统进程
        Slog.w(TAG, "*** GOODBYE!");
        Process.killProcess(Process.myPid());
        System.exit(10);
    } // end of while (true)

}
```

Watchdog运行后，便开始无限循环，依次调用每一个HandlerChecker的scheduleCheckLocked()方法
调度完HandlerChecker之后，便开始定期检查是否超时，每一次检查的间隔时间由CHECK_INTERVAL常量设定，为30秒
每一次检查都会调用evaluateCheckerCompletionLocked()方法来评估一下HandlerChecker的完成状态：
- COMPLETED表示已经完成
- WAITING和WAITED_HALF表示还在等待，但未超时
- OVERDUE表示已经超时。默认情况下，timeout是1分钟，但监测对象可以通过传参自行设定，譬如PKMS的Handler Checker的超时是10分钟
如果超时时间到了，还有HandlerChecker处于未完成的状态(OVERDUE)，则通过getBlockedCheckersLocked()方法，获取阻塞的HandlerChecker，生成一些描述信息
保存日志，包括一些运行时的堆栈信息，这些日志是我们解决Watchdog问题的重要依据。如果判断需要杀掉system_server进程，则给当前进程(system_server)发送signal 9
只要Watchdog没有发现超时的任务，HandlerChecker就会被不停的调度
```
public final class HandlerChecker implements Runnable {

    public void scheduleCheckLocked() {
        // Looper Checker中是不包含monitor对象的，判断消息队列是否处于空闲
        if (mMonitors.size() == 0 && mHandler.getLooper().isIdling()) {
            mCompleted = true;
            return;
        }
        ...
        // 将Monitor Checker的对象置于消息队列之前，优先运行
        mHandler.postAtFrontOfQueue(this);
    }

    @Override
    public void run() {
        // 依次调用Monitor对象的monitor()方法
        for (int i = 0 ; i < size ; i++) {
            synchronized (Watchdog.this) {
                mCurrentMonitor = mMonitors.get(i);
            }
            mCurrentMonitor.monitor();
        }
        ...
    }
}
```
对于Looper Checker而言，会判断线程的消息队列是否处于空闲状态。 如果被监测的消息队列一直闲不下来，则说明可能已经阻塞等待了很长时间
对于Monitor Checker而言，会调用实现类的monitor方法，譬如上文中提到的AMS.monitor()方法， 方法实现一般很简单，就是获取当前类的对象锁，如果当前对象锁已经被持有，则monitor()会一直处于wait状态，直到超时，这种情况下，很可能是线程发生了死锁