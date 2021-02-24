---
title: '[译文]Android内存泄漏的八种可能(上)'
date: 2019-02-26 09:58:40
categories: Android Framework
tags: [Android]
toc: true
description: How many cookies could a good cook cook if a good cook could cook cookies? A good cook could cook as much cookies as a good cook who could cook cookies.
---

Java是垃圾回收语言的一种，其优点是开发者无需特意管理内存分配，降低了应用由于局部故障(segmentation fault)导致崩溃，同时防止未释放的内存把堆栈(heap)挤爆的可能，所以写出来的代码更为安全。

不幸的是，在Java中仍存在很多容易导致内存泄漏的逻辑可能(logical leak)。如果不小心，你的Android应用很容易浪费掉未释放的内存，最终导致内存用光的错误抛出(out-of-memory，OOM)。

一般内存泄漏(traditional memory leak)的原因是：由忘记释放分配的内存导致的。（译者注：`Cursor`忘记关闭等）
逻辑内存泄漏(logical memory leak)的原因是：当应用不再需要这个对象，当仍未释放该对象的所有引用。

`如果持有对象的强引用，垃圾回收器是无法在内存中回收这个对象。`

在Android开发中，最容易引发的内存泄漏问题的是[Context](https://developer.android.com/reference/android/content/Context.html)。比如[Activity](https://developer.android.com/reference/android/app/Activity.html)的`Context`，就包含大量的内存引用，例如`View Hierarchies`和其他资源。一旦泄漏了`Context`，也意味泄漏它指向的所有对象。Android机器内存有限，太多的内存泄漏容易导致OOM。

检测逻辑内存泄漏需要主观判断，特别是对象的生命周期并不清晰。幸运的是，Activity有着明确的[生命周期](https://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle)，很容易发现泄漏的原因。[Activity.onDestroy()](https://developer.android.com/reference/android/app/Activity.html#onDestroy())被视为`Activity`生命的结束，程序上来看，它应该被销毁了，或者Android系统需要回收这些内存（译者注：当内存不够时，Android会回收看不见的`Activity`）。
如果这个方法执行完，在堆栈中仍存在持有该Activity的强引用，垃圾回收器就无法把它标记成已回收的内存，而我们本来目的就是要回收它！
结果就是`Activity`存活在它的生命周期之外。

`Activity`是重量级对象，应该让Android系统来处理它。然而，逻辑内存泄漏总是在不经意间发生。（译者注：曾经试过一个Activity导致20M内存泄漏）。在Android中，导致潜在内存泄漏的陷阱不外乎两种：

- 全局进程(process-global)的static变量。这个无视应用的状态，持有Activity的强引用的怪物。
- 活在Activity生命周期之外的线程。没有清空对Activity的强引用。

检查一下你有没有遇到下列的情况。

# Static Activities

在类中定义了静态`Activity`变量，把当前运行的`Activity`实例赋值于这个静态变量。
如果这个静态变量在`Activity`生命周期结束后没有清空，就导致内存泄漏。因为static变量是贯穿这个应用的生命周期的，所以被泄漏的`Activity`就会一直存在于应用的进程中，不会被垃圾回收器回收。

```java
  static Activity activity;

    void setStaticActivity() {
      activity = this;
    }

    View saButton = findViewById(R.id.sa_button);
    saButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticActivity();
        nextActivity();
      }
    });
```

![Memory Leak 1 - Static Activity](https://i.loli.net/2019/02/26/5c74a0feb907c.png)

# Static Views

类似的情况会发生在单例模式中，如果`Activity`经常被用到，那么在内存中保存一个实例是很实用的。正如之前所述，强制延长`Activity`的生命周期是相当危险而且不必要的，无论如何都不能这样做。

特殊情况：如果一个View初始化耗费大量资源，而且在一个`Activity`生命周期内保持不变，那可以把它变成static，加载到视图树上(View Hierachy)，[像这样](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L132)，当`Activity`被销毁时，应当释放资源。（译者注：示例代码中并没有释放内存，把这个static view置null即可，但是还是不建议用这个static view的方法）

```java
 static view;

    void setStaticView() {
      view = findViewById(R.id.sv_button);
    }

    View svButton = findViewById(R.id.sv_button);
    svButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
        setStaticView();
        nextActivity();
      }
    });
```

![Memory Leak 2 - Static View](https://i.loli.net/2019/02/26/5c74a28acb265.png)

# Inner Classes

继续，假设`Activity`中有个[内部类](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L126)，这样做可以提高可读性和封装性。将如我们创建一个内部类，而且持有一个静态变量的引用，恭喜，内存泄漏就离你不远了（译者注：销毁的时候置空，嗯）。

```java
 private static Object inner;

       void createInnerClass() {
        class InnerClass {
        }
        inner = new InnerClass();
    }

    View icButton = findViewById(R.id.ic_button);
    icButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            createInnerClass();
            nextActivity();
        }
    });
```

![Memory Leak 3 - Inner Class](https://i.loli.net/2019/02/26/5c74a3827f856.png)

内部类的优势之一就是可以访问外部类，不幸的是，导致内存泄漏的原因，就是内部类持有外部类实例的强引用。

# Anonymous Classes

相似地，匿名类也维护了外部类的引用。所以内存泄漏很容易发生，[当你在`Activity`中定义了匿名的`AsyncTask`](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L102)
。当异步任务在后台执行耗时任务期间，`Activity`不幸被销毁了（译者注：用户退出，系统回收），这个被`AsyncTask`持有的`Activity`实例就不会被垃圾回收器回收，直到异步任务结束。

```java
  void startAsyncTask() {
        new AsyncTask<Void, Void, Void>() {
            @Override protected Void doInBackground(Void... params) {
                while(true);
            }
        }.execute();
    }

    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    View aicButton = findViewById(R.id.at_button);
    aicButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            startAsyncTask();
            nextActivity();
        }
    });
```

![Memory Leak 4 - AsyncTask](https://i.loli.net/2019/02/26/5c74a4bd45062.png)

# Handler

同样道理，[定义匿名的`Runnable`，用匿名类`Handler`执行](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L114)。`Runnable`内部类会持有外部类的隐式引用，被传递到`Handler`的消息队列`MessageQueue`中，在`Message`消息没有被处理之前，`Activity`实例不会被销毁了，于是导致内存泄漏。

```java
void createHandler() {
        new Handler() {
            @Override public void handleMessage(Message message) {
                super.handleMessage(message);
            }
        }.postDelayed(new Runnable() {
            @Override public void run() {
                while(true);
            }
        }, Long.MAX_VALUE >> 1);
    }

    View hButton = findViewById(R.id.h_button);
    hButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            createHandler();
            nextActivity();
        }
    });
```

![Memory Leak 5 - Handler](https://i.loli.net/2019/02/26/5c74a5ab1f03e.png)

# Threads

我们再次通过[Thread](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L142)和[TimerTask](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L150)来展现内存泄漏。

```java
  void spawnThread() {
        new Thread() {
            @Override public void run() {
                while(true);
            }
        }.start();
    }

    View tButton = findViewById(R.id.t_button);
    tButton.setOnClickListener(new View.OnClickListener() {
      @Override public void onClick(View v) {
          spawnThread();
          nextActivity();
      }
    });
```

![Memory Leak 6 - Thread](https://i.loli.net/2019/02/26/5c74a695b7543.png)

# TimerTask

只要是匿名类的实例，不管是不是在工作线程，都会持有`Activity`的引用，导致内存泄漏。

```java
void scheduleTimer() {
        new Timer().schedule(new TimerTask() {
            @Override
            public void run() {
                while(true);
            }
        }, Long.MAX_VALUE >> 1);
    }

    View ttButton = findViewById(R.id.tt_button);
    ttButton.setOnClickListener(new View.OnClickListener() {
        @Override public void onClick(View v) {
            scheduleTimer();
            nextActivity();
        }
    });
```

![Memory Leak 7 - TimerTask](https://i.loli.net/2019/02/26/5c74a70924397.png)

# Sensor Manager

最后，通过[Context.getSystemService(int name)](https://developer.android.com/reference/android/content/Context.html#getSystemService(java.lang.String))可以获取系统服务。这些服务工作在各自的进程中，帮助应用处理后台任务，处理硬件交互。如果需要使用这些服务，可以注册[监听器](https://github.com/NimbleDroid/Memory-Leaks/blob/master/app/src/main/java/com/nimbledroid/memoryleaks/MainActivity.java#L136)，这会导致服务持有了`Context`的引用，如果在`Activity`销毁的时候没有注销这些监听器，会导致内存泄漏。

```java
 void registerListener() {
               SensorManager sensorManager = (SensorManager) getSystemService(SENSOR_SERVICE);
               Sensor sensor = sensorManager.getDefaultSensor(Sensor.TYPE_ALL);
               sensorManager.registerListener(this, sensor, SensorManager.SENSOR_DELAY_FASTEST);
        }

        View smButton = findViewById(R.id.sm_button);
        smButton.setOnClickListener(new View.OnClickListener() {
            @Override public void onClick(View v) {
                registerListener();
                nextActivity();
            }
        });
```

![Memory Leak 8 - Sensor Manager](https://i.loli.net/2019/02/26/5c74a7d1725c1.png)

# 总结

看过那么多会导致内存泄漏的例子，容易导致吃光手机的内存使垃圾回收处理更为频发，甚至最坏的情况会导致OOM。垃圾回收的操作是很昂贵的开销，会导致肉眼可见的卡顿。所以，实例化的时候注意持有的引用链，并经常进行内存泄漏检查。

（译者注：本文没有提及比较通用的解决方法，[具体可以参考这篇文章，静态内部类解脱你的烦恼](https://xmaihh.github.io/2019/02/26/%E8%AF%91%E6%96%87-Android%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E7%9A%84%E5%85%AB%E7%A7%8D%E5%AF%B9%E5%BA%94%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95-%E4%B8%8B/)）

祝好运。

# Reference

[http://blog.nimbledroid.com/2016/05/23/memory-leaks.html](http://blog.nimbledroid.com/2016/05/23/memory-leaks.html)