---
title: ANR程序问题分析之dropbox
date: 2018-07-14 18:39:01
categories: Android Framework
tags: [Android]
toc: true
description: A bitter end is much better than bitterness without an end. 
---
从2.2开始增加了DropBox功能，增强Android的异常信息收集管理能力DropBox（简称DB）是系统进程中的一个服务，在system_server进程启动时创建，并且它没有运行在单独的线程中，而是运行在system_server的ServerThread线程中。我们可以将ServerThread称作system_server的主线程，ServerThread线程除了启动并维护各个服务外，还负责检测一些重要的服务是否死锁
DropBoxManagerService（简称DBMS）就是DB服务的本尊，它的主要功能接口包括以下几个函数：
public voidadd(DropBoxManager.Entryentry)

DBMS将所有要添加的日志都用DropBoxManager.Entry类型的对象表示，通过add函数添加，并且直到目前为止一个Entry对象对应着一个日志文件。

publicboolean isTagEnabled(String tag)

通过给每一个Entry设置一个tag可以标识不同类型的日志，并且可以灵活的启用/禁用某种类型的日志，isTagEnabled用来判断指定类型的日志是否被启用/禁用了，一旦禁用就不会再记录这种类型的日志。默认是不禁用任何类型的日志的。稍后说明如何启用/禁用日志。

publicsynchronized DropBoxManager.Entry getNextEntry(String tag, long millis)

我们可以通过getNextEntry函数获取指定类型和指定时间点之后的第一条日志，要使用这个功能应用程序需要有“android.permission.READ_LOGS”的权限，并且在使用完毕返回的Entry对象后要调用其close函数确保关闭日志文件的文件描述符（如果不关闭的话可能造成进程打开的文件描述符超过1024而崩溃，Android中限制每个进程的文件描述符上限为1024）。

DBMS提供了很多的配置项用来限制对磁盘的使用，通过SettingsProvider应用程序维护，数据存放在其settings.db数据库中。这些配置项也都有默认值，罗列如下：
```
Settings.Secure.DROPBOX_AGE_SECONDS = "dropbox_age_seconds"
//日志文件保存的最长时间，默认3天
Settings.Secure.DROPBOX_MAX_FILES = "dropbox_max_files"
//日志文件的最大数量，默认值是1000
Settings.Secure.DROPBOX_QUOTA_KB = "dropbox_quota_kb"
//磁盘空间最大使用量
Settings.Secure.DROPBOX_QUOTA_PERCENT = "dropbox_quota_percent"
Settings.Secure.DROPBOX_RESERVE_PERCENT = "dropbox_reserve_percent"
Settings.Secure.DROPBOX_TAG_PREFIX = "dropbox:"
//应用程序可以利用DropBox来做事情，收集日志等
```
### DropBox启动
在SystemServer.java的ServerThread.run()里
```
Slog.i(TAG, "DropBox Service");
ServiceManager.addService(Context.DROPBOX_SERVICE, //服务名称为“dropbox”
new DropBoxManagerService(context, new File("/data/system/dropbox")));
```
DROPBOX_SERVICE = “dropbox”, "/data/system/dropbox"是DB指定的文件存放位置，这个过程向ServiceManager 登记名为“dropbox”的服务。那么可通过`dumpsys dropbox`来查看该dropbox服务信息。
### DropBox初始化
在DropBoxManagerService.java的构造方法里
```
public final class DropBoxManagerService extends IDropBoxManagerService.Stub {

    public DropBoxManagerService(final Context context, File path) {
        mDropBoxDir = path;  // 目录/data/system/dropbox
        mContext = context;
        mContentResolver = context.getContentResolver();

        IntentFilter filter = new IntentFilter();
        // 监听存储设备可用空间低的广播
        filter.addAction(Intent.ACTION_DEVICE_STORAGE_LOW);
        // 监听开机完毕的广播
        filter.addAction(Intent.ACTION_BOOT_COMPLETED);
        context.registerReceiver(mReceiver, filter);

        // Settings数据库变化时则回调广播接收者的onReceive方法,此处CONTENT_URI=content://settings/global"
        mContentResolver.registerContentObserver(
            Settings.Global.CONTENT_URI, true,
            new ContentObserver(new Handler()) {
                public void onChange(boolean selfChange) {
                    mReceiver.onReceive(context, (Intent) null);
                }
            });

        mHandler = new Handler() {
            public void handleMessage(Message msg) {
                // 发送广播
                if (msg.what == MSG_SEND_BROADCAST) {
                    mContext.sendBroadcastAsUser((Intent)msg.obj, UserHandle.OWNER,
                            android.Manifest.permission.READ_LOGS);
                }
            }
        };
    }
}
```
该方法主要功能是给dropbox目录所对应的存储空间进行瘦身:

- 存储设备可用空间低；
- 开机完毕；
- Settings数据库变化；
以上情况都会触发触发执行mReceiver的onReceive方法
在DropBoxManagerService.java的mReceiver
```
private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
    public void onReceive(Context context, Intent intent) {
        if (intent != null && Intent.ACTION_BOOT_COMPLETED.equals(intent.getAction())) {
            mBooted = true;
            return;
        }

        //收到ACTION_DEVICE_STORAGE_LOW，则强制重新check存储空间
        mCachedQuotaUptimeMillis = 0; 

        //创建工作线程来执行init和trim操作
        new Thread() {
            public void run() {
                try {
                    init(); //初始化
                    trimToFit(); //清除一些空间
                } catch (IOException e) {
                    ...
                }
            }
        }.start();
    }
};
```
#### init()方法
```
private synchronized void init() throws IOException {
    if (mStatFs == null) {
        if (!mDropBoxDir.isDirectory() && !mDropBoxDir.mkdirs()) {
            ...
        }
        mStatFs = new StatFs(mDropBoxDir.getPath());
        mBlockSize = mStatFs.getBlockSize(); //mBlockSize=4096
    }

    if (mAllFiles == null) {
        File[] files = mDropBoxDir.listFiles();
        // 列举所有的dropbox文件
        mAllFiles = new FileList();
        mFilesByTag = new HashMap<String, FileList>();

        for (File file : files) {
            if (file.getName().endsWith(".tmp")) {
                file.delete(); //删除后缀为.tmp文件
                continue;
            }
            // 创建dropbox的实体文件对象, 根据文件名来获取相应的时间戳
            EntryFile entry = new EntryFile(file, mBlockSize);
            if (entry.tag == null) {
                continue; //忽略tag为空的文件
            } else if (entry.timestampMillis == 0) {
                file.delete(); //删除时间戳为0的文件
                continue;
            }
            //将entry加入到mAllFiles对象
            enrollEntry(entry);
        }
    }
}
```
该方法主要功能：

- 创建目录/data/system/dropbox;
- 将每一个dropbox文件都对应于一个EntryFile对象,根据文件名来获取相应的时间戳
- 删除后缀为.tmp的文件;
- 删除时间戳为0的文件
#### trimToFit()方法
```
private synchronized long trimToFit() {
    int ageSeconds = Settings.Global.getInt(mContentResolver,
            Settings.Global.DROPBOX_AGE_SECONDS, DEFAULT_AGE_SECONDS);
    int maxFiles = Settings.Global.getInt(mContentResolver,
            Settings.Global.DROPBOX_MAX_FILES, DEFAULT_MAX_FILES);
    long cutoffMillis = System.currentTimeMillis() - ageSeconds * 1000;
    
    while (!mAllFiles.contents.isEmpty()) {
        EntryFile entry = mAllFiles.contents.first();
        //当最老的文件时间戳在3天之内，且文件个数低于1000，则跳出循环
        if (entry.timestampMillis > cutoffMillis 
            && mAllFiles.contents.size() < maxFiles) break;
        FileList tag = mFilesByTag.get(entry.tag);
        if (tag != null && tag.contents.remove(entry)) tag.blocks -= entry.blocks;
        if (mAllFiles.contents.remove(entry)) mAllFiles.blocks -= entry.blocks;
        if (entry.file != null) entry.file.delete(); //删除文件
    }
    long uptimeMillis = SystemClock.uptimeMillis();
    //除非接收设备存储低的广播，否则间隔5s才能再次执行restat
    if (uptimeMillis > mCachedQuotaUptimeMillis + QUOTA_RESCAN_MILLIS) {
        int quotaPercent = Settings.Global.getInt(mContentResolver,
                Settings.Global.DROPBOX_QUOTA_PERCENT, DEFAULT_QUOTA_PERCENT);
        int reservePercent = Settings.Global.getInt(mContentResolver,
                Settings.Global.DROPBOX_RESERVE_PERCENT, DEFAULT_RESERVE_PERCENT);
        int quotaKb = Settings.Global.getInt(mContentResolver,
                Settings.Global.DROPBOX_QUOTA_KB, DEFAULT_QUOTA_KB);
        //重新统计文件
        mStatFs.restat(mDropBoxDir.getPath());
        int available = mStatFs.getAvailableBlocks();
        int nonreserved = available - mStatFs.getBlockCount() * reservePercent / 100;
        int maximum = quotaKb * 1024 / mBlockSize;
        //可用的块数量
        mCachedQuotaBlocks = Math.min(maximum, Math.max(0, nonreserved * quotaPercent / 100));
        mCachedQuotaUptimeMillis = uptimeMillis;
    }
    if (mAllFiles.blocks > mCachedQuotaBlocks) {
        //公平地限制所有tag的空间
        int unsqueezed = mAllFiles.blocks, squeezed = 0;
        TreeSet<FileList> tags = new TreeSet<FileList>(mFilesByTag.values());
        for (FileList tag : tags) {
            if (squeezed > 0 && tag.blocks <= (mCachedQuotaBlocks - unsqueezed) / squeezed) {
                break;
            }
            unsqueezed -= tag.blocks;
            squeezed++;
        }
        int tagQuota = (mCachedQuotaBlocks - unsqueezed) / squeezed;
        //移除每个tags中的旧items
        for (FileList tag : tags) {
            if (mAllFiles.blocks < mCachedQuotaBlocks) break;
            while (tag.blocks > tagQuota && !tag.contents.isEmpty()) {
                EntryFile entry = tag.contents.first();
                if (tag.contents.remove(entry)) tag.blocks -= entry.blocks;
                if (mAllFiles.contents.remove(entry)) mAllFiles.blocks -= entry.blocks;
                try {
                    if (entry.file != null) entry.file.delete();
                    enrollEntry(new EntryFile(mDropBoxDir, entry.tag, entry.timestampMillis));
                } catch (IOException e) {
                    Slog.e(TAG, "Can't write tombstone file", e);
                }
            }
        }
    }
    return mCachedQuotaBlocks * mBlockSize;
}
```
trimToFit过程中触发条件是：当文件有效时长超过3天，或者最大文件数超过1000，再或者剩余可用存储设备过低；
DBMS有很多常量参数：

- DEFAULT_AGE_SECONDS = 3 * 86400：文件最长可存活时长为3天
- DEFAULT_MAX_FILES = 1000：最大dropbox文件个数为1000
- DEFAULT_QUOTA_KB = 5 * 1024：分配dropbox空间的最大值5M
- DEFAULT_QUOTA_PERCENT = 10：是指dropbox目录最多可占用空间比例10%
- DEFAULT_RESERVE_PERCENT = 10：是指dropbox不可使用的存储空间比例10%
- QUOTA_RESCAN_MILLIS = 5000：重新扫描retrim时长为5s
当然上面这些都是默认值，完全可以通过设置content://settings/global数据库中相应项来设定值。

### DropBox工作
以下任一场景，都会调用AMS.addErrorToDropBox()来触发DBMS工作。

- crash: AMS.handleApplicationCrashInner()
- anr: AMS.appNotResponding()
- watchdog: Watchdog.run()
- native_crash: NativeCrashReporter.run()
- wtf: 当调用Log.wtf()或者Log.wtfQuiet()
- lowmem: 当内存较低时，触发AMS.reportMemUsage()

在ActivityManagerService.java的addErrorToDropBox()方法
```
public void addErrorToDropBox(String eventType, ProcessRecord process, String processName, ActivityRecord activity, ActivityRecord parent, String subject, final String report, final File logFile, final ApplicationErrorReport.CrashInfo crashInfo) {

    //创建dropbox标签名
    final String dropboxTag = processClass(process) + "_" + eventType;
    //获取dropbox服务的代理端
    final DropBoxManager dbox = (DropBoxManager)
            mContext.getSystemService(Context.DROPBOX_SERVICE);

    //当不需要输出dropbox报告则直接返回
    if (dbox == null || !dbox.isTagEnabled(dropboxTag)) return;

    final StringBuilder sb = new StringBuilder(1024);
    //输出Process,flags,以及进程中所有package
    appendDropBoxProcessHeaders(process, processName, sb);
    ...
    if (subject != null) {
        sb.append("Subject: ").append(subject).append("\n");
    }
    sb.append("Build: ").append(Build.FINGERPRINT).append("\n");
    sb.append("\n");

    //创建新线程，避免将调用者阻塞在I/O
    Thread worker = new Thread("Error dump: " + dropboxTag) {
        @Override
        public void run() {
            if (report != null) {
                //比如ANR时输出Cpuinfo，或者lowmem时输出的内存信息
                sb.append(report); 
            }
            if (logFile != null) {
                //比如anr或者Watchdog时输出的traces文件(kill -3)，最大上限为256KB
                sb.append(FileUtils.readTextFile(logFile, DROPBOX_MAX_SIZE,
                            "\n\n[[TRUNCATED]]"));
            }
            if (crashInfo != null && crashInfo.stackTrace != null) {
                // 比如crash时输出的调用栈
                sb.append(crashInfo.stackTrace);
            }

            String setting = Settings.Global.ERROR_LOGCAT_PREFIX + dropboxTag;
            int lines = Settings.Global.getInt(mContext.getContentResolver(), setting, 0);
            //当dropboxTag所对应的settings项不等于0，则输出logcat
            if (lines > 0) {
              //输出evets/system/main/crash这些log信息
              java.lang.Process logcat = new ProcessBuilder("/system/bin/logcat",
                      "-v", "time", "-b", "events", "-b", "system", "-b", "main",
                      "-b", "crash",
                      "-t", String.valueOf(lines)).redirectErrorStream(true).start();

              input = new InputStreamReader(logcat.getInputStream());

              int num;
              char[] buf = new char[8192];
              //不断读取input中的log内容，并添加到sb
              while ((num = input.read(buf)) > 0) sb.append(buf, 0, num);
              ...
            }
            //将log信息输出到DropBox
            dbox.addText(dropboxTag, sb.toString());
        }
    };

    if (process == null) {
        //当进程为空，意味着system_server进程崩溃，系统可能很快就要挂了,
        //那么不再创建新线程，而是直接在system_server进程中同步运行
        worker.run();
    } else {
        //启动新线程
        worker.start();
    }
}
```
该方法主要功能是输出以下内容项：

- Process,flags, package等头信息；
- 当report不为空，则比如ANR时输出Cpuinfo，或者lowmem时输出的内存信息
- 当logFile不为空，则比如anr或者Watchdog时输出的traces文件(kill -3)，最大上限为256KB；
- 当stack不为空，则比如crash时输出的调用栈；
- 输出logcat的events/system/main/crash信息。

AMS.processClass()方法
```
private static String processClass(ProcessRecord process) {
    //MY_PID代表的是当前进程pid，正是system_server进程
    if (process == null || process.pid == MY_PID) {
        return "system_server";
    } else if ((process.info.flags & ApplicationInfo.FLAG_SYSTEM) != 0) {
        return "system_app";
    } else {
        return "data_app";
    }
}
```
dropbox文件名格式为dropboxTag@xxx.txt xxx代表时间戳,例如system_server_crash@1465650845355.txt,则记录该文件时间戳为1465650845355. 文件后缀除了.txt，还有压缩格式.txt.gz. 对于dropboxTag是由processClass + eventType组合而成.

processClass分为system_server, system_app, data_app;
eventType：分为crash,anr,wtf,native_cras,lowmem, watchdog

列举部分常见tags以及含义:

| dropboxTag | 含义 |
| --- | --- |
| system_server_anr | system进程无响应 |
| system_server_watchdog | system进程发生watchdog |
| system_server_crash | system进程崩溃 |
| system_server_native_crash | system进程native出现崩溃 |
| system_server_wtf | system进程发生严重错误 |
| system_server_lowmem | system进程内存不足 |
当然除了system_server进程, 还有system_app, data_app类型的进程, 以上所有类型都适用,列举部分:

| dropboxTag | 含义 |
| --- | --- |
| system_app_crash | 系统app崩溃 |
| system_app_anr | 系统app无响应 |
| data_app_crash | 普通app崩溃 |
| data_app_anr | 普通app无响应 |
