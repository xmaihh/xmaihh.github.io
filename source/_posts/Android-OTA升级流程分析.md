---
title: Android OTA升级流程分析
date: 2018-07-24 21:23:26
categories: Android Framework
tags: [Android]
toc: true
description: And glory like the phoenix midst her fires,.Exhales her odours, blazes, and expires. 
---
Android系统Recovery使用update.zip升级过程分析,update.zip包来源有两种，一个是OTA在线下载（一般下载到/CACHE分区），一个是手动拷贝到T卡
这里分析从update.zip拷贝到T卡后，弹出升级对话框分析：
## 重启至recovery
mNowButton按钮的监听事件里，会调用mService.rebootAndUpdate（new File(mFile)）。这个mService就是SystemUpdateService的实例
这个mFile就是update.zip的路径。
### 1.调用RecoverySystem类提供的verifyPackage方法进行签名验证
签名验证函数，实现过程就不贴出来了，
参数，
- packageFile--升级文件
- listener--进度监督器
- deviceCertsZipFile--签名文件，如果为空，则使用系统默认的签名
只有当签名验证正确才返回，否则将抛出异常。
在Recovery模式下进行升级时候也是会进行签名验证的，如果这里先不进行验证也不会有什么问题。但是我们建议在重启前，先验证，以便及早发现问题。
如果签名验证没有问题，就执行installPackage开始升级。
在 recovery 模式升级的时候。我们recovery模式会再一次验证update.zip签名。具体代码在install.cpp 中函数 really_install_package
```
    public static void verifyPackage(File packageFile,
                                     ProgressListener listener,
                                     File deviceCertsZipFile)
        throws IOException, GeneralSecurityException
```
### 2.签名验证没有问题，就进行重启升级
installPackage(Context context,FilepackageFile)函数根据我们传过来的包文件，获取这个包文件的绝对路径filename。然后将其拼成arg=“--update_package=”+filename。它最终会被写入到BCB中。这个就是重启进入Recovery模式后，Recovery服务要进行的操作。它被传递到函数bootCommand(context,arg)
```
    public static void installPackage(Context context, File packageFile)
        throws IOException {
        String filename = packageFile.getCanonicalPath();
        Log.w(TAG, "!!! REBOOTING TO INSTALL " + filename + " !!!");
 
        final String filenameArg = "--update_package=" + filename;
        final String localeArg = "--locale=" + Locale.getDefault().toString();
        bootCommand(context, filenameArg, localeArg);
    }
```
bootCommand()函数创建/cache/recovery/目录，删除这个目录下的command和log（如果存在）文件在sqlite数据库中的备份。然后将上一步中的arg命令写入到/cache/recovery/command文件中。下一步就是真正重启了。接下来看一下在重启函数reboot中所做的事情
```
    private static void bootCommand(Context context, String... args) throws IOException {
        RECOVERY_DIR.mkdirs();  // In case we need it
        COMMAND_FILE.delete();  // In case it's not writable
        LOG_FILE.delete();
 
        FileWriter command = new FileWriter(COMMAND_FILE);
        try {
            for (String arg : args) {
                if (!TextUtils.isEmpty(arg)) {
                    command.write(arg);
                    command.write("\n");
                }
            }
        } finally {
            command.close();
        }
 
        // Having written the command file, go ahead and reboot
        PowerManager pm = (PowerManager) context.getSystemService(Context.POWER_SERVICE);
        pm.reboot(PowerManager.REBOOT_RECOVERY);
 
        throw new IOException("Reboot failed (no permissions?)");
    }
```
### 4.系统重启进入Recovery模式
pm.reboot()重启之前先获得了PowerManager（电源管理）并进一步获得其系统服务。然后调用了pm.reboot(“recovery”)函数。他就是./bionic/libc/unistd/reboot.c中的reboot函数。这个函数实际上是一个系统调用，即__reboot(LINUX_REBOOT_MAGIC1,LINUX_REBOOT_MAGIC2,mode,NULL);mode就是我们传过来的“recovery”
进入Recovery模式
- 将"boot-recovery"写入BCB的command域
- 将"--update_package=/cache/update.zip"或则"--update_package=/sdcard/update.zip"写入/cache/recovery/command文件中
系统重启时会判断/cache/recovery目录下是否有command文件，如果存在就进入recovery模式，否则就正常启动
进入到Recovery模式下，将执行recovery.cpp的main函数
```
static const struct option OPTIONS[] = {
  { "send_intent", required_argument, NULL, 's' },
  { "update_package", required_argument, NULL, 'u' },
  { "wipe_data", no_argument, NULL, 'w' },
  { "wipe_cache", no_argument, NULL, 'c' },
  { "show_text", no_argument, NULL, 't' },
  { "just_exit", no_argument, NULL, 'x' },
  { "locale", required_argument, NULL, 'l' },
  { "stages", required_argument, NULL, 'g' },
  { "shutdown_after", no_argument, NULL, 'p' },
  { "reason", required_argument, NULL, 'r' },
  { NULL, 0, NULL, 0 },
};
```
在这个While循环，用来读取recovery的command参数，OPTIONS的不同选项定义
写入的命令文件内容，将为update_package 赋值
```
    if (update_package) {
        // For backwards compatibility on the cache partition only, if
        // we're given an old 'root' path "CACHE:foo", change it to
        // "/cache/foo".
        if (strncmp(update_package, "CACHE:", 6) == 0) {
            int len = strlen(update_package) + 10;
            char* modified_path = (char*)malloc(len);
            strlcpy(modified_path, "/cache/", len);
            strlcat(modified_path, update_package+6, len);
            printf("(replacing path \"%s\" with \"%s\")\n",
                   update_package, modified_path);
            update_package = modified_path;
        }
    }
```
### 5.install.cpp进行升级操作
具体的升级过程都是在install.cpp中执行的，先看install_package方法
```
int
install_package(const char* path, int* wipe_cache, const char* install_file,
                bool needs_mount)
{
    FILE* install_log = fopen_path(install_file, "w");
    if (install_log) {
        fputs(path, install_log);
        fputc('\n', install_log);
    } else {
        LOGE("failed to open last_install: %s\n", strerror(errno));
    }
    int result;
    if (setup_install_mounts() != 0) {
        LOGE("failed to set up expected mounts for install; aborting\n");
        result = INSTALL_ERROR;
    } else {
        result = really_install_package(path, wipe_cache, needs_mount);
    }
    if (install_log) {
        fputc(result == INSTALL_SUCCESS ? '1' : '0', install_log);
        fputc('\n', install_log);
        fclose(install_log);
    }
    return result;
}
...
static int
really_install_package(const char *path, int* wipe_cache, bool needs_mount)
{
    ui->SetBackground(RecoveryUI::INSTALLING_UPDATE);
    ui->Print("Finding update package...\n");
    // Give verification half the progress bar...
    ui->SetProgressType(RecoveryUI::DETERMINATE);
    ui->ShowProgress(VERIFICATION_PROGRESS_FRACTION, VERIFICATION_PROGRESS_TIME);
    LOGI("Update location: %s\n", path);
 
    // Map the update package into memory.
    ui->Print("Opening update package...\n");
 
    if (path && needs_mount) {
        if (path[0] == '@') {
            ensure_path_mounted(path+1);
        } else {
            ensure_path_mounted(path);
        }
    }
 
    MemMapping map;
    if (sysMapFile(path, &map) != 0) {
        LOGE("failed to map file\n");
        return INSTALL_CORRUPT;
    }
 
    // 装入签名文件
    int numKeys;
    Certificate* loadedKeys = load_keys(PUBLIC_KEYS_FILE, &numKeys);
    if (loadedKeys == NULL) {
        LOGE("Failed to load keys\n");
        return INSTALL_CORRUPT;
    }
    LOGI("%d key(s) loaded from %s\n", numKeys, PUBLIC_KEYS_FILE);
 
    ui->Print("Verifying update package...\n");
 
    // 验证签名
    int err;
    err = verify_file(map.addr, map.length, loadedKeys, numKeys);
    free(loadedKeys);
    LOGI("verify_file returned %d\n", err);
    // 签名失败的处理
    if (err != VERIFY_SUCCESS) {
        LOGE("signature verification failed\n");
        sysReleaseMap(&map);
        return INSTALL_CORRUPT;
    }
 
    /* Try to open the package.
     */
    // 打开升级包
    ZipArchive zip;
    err = mzOpenZipArchive(map.addr, map.length, &zip);
    if (err != 0) {
        LOGE("Can't open %s\n(%s)\n", path, err != -1 ? strerror(err) : "bad");
        sysReleaseMap(&map);
        return INSTALL_CORRUPT;
    }
 
    /* Verify and install the contents of the package.
     */
    ui->Print("Installing update...\n");
    ui->SetEnableReboot(false);
    // 执行升级脚本文件，开始升级
    int result = try_update_binary(path, &zip, wipe_cache);
    ui->SetEnableReboot(true);
    ui->Print("\n");
 
    sysReleaseMap(&map);
 
    return result;
}

```
在这里创建了log文件，升级过程包括出错的信息都会写到这个文件中，便于后续的分析工作,
接下来是really_install_package里依次
(1)验证签名，装载签名文件，如果为空 ，终止升级； 调用verify_file进行签名验证，这个方法定义在verifier.cpp文件中，此处不展开，如果验证失败立即终止升级。
(2)读取升级包信息,执行mzOpenZipArchive方法，打开升级包并扫描，将包的内容拷贝到变量zip中，该变量将作为参数用来执行升级脚本
(3)执行升级脚本文件，开始升级, try_update_binary方法用来处理升级包，执行制作升级包中的脚本文件update_binary，进行系统更新
### 6.try_update_binary执行升级脚本
```
// If the package contains an update binary, extract it and run it.
static int
try_update_binary(const char *path, ZipArchive *zip, int* wipe_cache) {
	// 检查update-binary是否存在
    const ZipEntry* binary_entry =
            mzFindZipEntry(zip, ASSUMED_UPDATE_BINARY_NAME);
    if (binary_entry == NULL) {
        mzCloseZipArchive(zip);
        return INSTALL_CORRUPT;
    }
 
    const char* binary = "/tmp/update_binary";
    unlink(binary);
    int fd = creat(binary, 0755);
    if (fd < 0) {
        mzCloseZipArchive(zip);
        LOGE("Can't make %s\n", binary);
        return INSTALL_ERROR;
    }
    // update-binary拷贝到"/tmp/update_binary"
    bool ok = mzExtractZipEntryToFile(zip, binary_entry, fd);
    close(fd);
    mzCloseZipArchive(zip);
 
    if (!ok) {
        LOGE("Can't copy %s\n", ASSUMED_UPDATE_BINARY_NAME);
        return INSTALL_ERROR;
    }
 
    // 创建管道，用于下面的子进程和父进程之间的通信
    int pipefd[2];
    pipe(pipefd);
 
    // When executing the update binary contained in the package, the
    // arguments passed are:
    //
    //   - the version number for this interface
    //
    //   - an fd to which the program can write in order to update the
    //     progress bar.  The program can write single-line commands:
    //
    //        progress <frac> <secs>
    //            fill up the next <frac> part of of the progress bar
    //            over <secs> seconds.  If <secs> is zero, use
    //            set_progress commands to manually control the
    //            progress of this segment of the bar
    //
    //        set_progress <frac>
    //            <frac> should be between 0.0 and 1.0; sets the
    //            progress bar within the segment defined by the most
    //            recent progress command.
    //
    //        firmware <"hboot"|"radio"> <filename>
    //            arrange to install the contents of <filename> in the
    //            given partition on reboot.
    //
    //            (API v2: <filename> may start with "PACKAGE:" to
    //            indicate taking a file from the OTA package.)
    //
    //            (API v3: this command no longer exists.)
    //
    //        ui_print <string>
    //            display <string> on the screen.
    //
    //   - the name of the package zip file.
    //
 
    const char** args = (const char**)malloc(sizeof(char*) * 5);
    args[0] = binary;
    args[1] = EXPAND(RECOVERY_API_VERSION);   // defined in Android.mk
    char* temp = (char*)malloc(10);
    sprintf(temp, "%d", pipefd[1]);
    args[2] = temp;
    args[3] = (char*)path;
    args[4] = NULL;
 
    // 创建子进程。负责执行binary脚本
    pid_t pid = fork();
    if (pid == 0) {
        umask(022);
        close(pipefd[0]);
        execv(binary, (char* const*)args);// 执行binary脚本
        fprintf(stdout, "E:Can't run %s (%s)\n", binary, strerror(errno));
        _exit(-1);
    }
    close(pipefd[1]);
 
    *wipe_cache = 0;
 
    // 父进程负责接受子进程发送的命令去更新ui显示
    char buffer[1024];
    FILE* from_child = fdopen(pipefd[0], "r");
    while (fgets(buffer, sizeof(buffer), from_child) != NULL) {
        char* command = strtok(buffer, " \n");
        if (command == NULL) {
            continue;
        } else if (strcmp(command, "progress") == 0) {
            char* fraction_s = strtok(NULL, " \n");
            char* seconds_s = strtok(NULL, " \n");
 
            float fraction = strtof(fraction_s, NULL);
            int seconds = strtol(seconds_s, NULL, 10);
 
            ui->ShowProgress(fraction * (1-VERIFICATION_PROGRESS_FRACTION), seconds);
        } else if (strcmp(command, "set_progress") == 0) {
            char* fraction_s = strtok(NULL, " \n");
            float fraction = strtof(fraction_s, NULL);
            ui->SetProgress(fraction);
        } else if (strcmp(command, "ui_print") == 0) {
            char* str = strtok(NULL, "\n");
            if (str) {
                ui->Print("%s", str);
            } else {
                ui->Print("\n");
            }
            fflush(stdout);
        } else if (strcmp(command, "wipe_cache") == 0) {
            *wipe_cache = 1;
        } else if (strcmp(command, "clear_display") == 0) {
            ui->SetBackground(RecoveryUI::NONE);
        } else if (strcmp(command, "enable_reboot") == 0) {
            // packages can explicitly request that they want the user
            // to be able to reboot during installation (useful for
            // debugging packages that don't exit).
            ui->SetEnableReboot(true);
        } else {
            LOGE("unknown command [%s]\n", command);
        }
    }
    fclose(from_child);
 
    int status;
    waitpid(pid, &status, 0);
    if (!WIFEXITED(status) || WEXITSTATUS(status) != 0) {
        LOGE("Error in %s\n(Status %d)\n", path, WEXITSTATUS(status));
        return INSTALL_ERROR;
    }
 
    return INSTALL_SUCCESS;
}
```
try_update_binary函数，是真正实现读取升级包中的脚本文件并执行相应的函数的地方。在此函数中，通过调用fork函数创建出一个子进程，在子进程中开始读取并执行升级脚本文件。在此需要注意的是函数fork的用法，fork被调用一次，将做两次返回，在父进程中返回的是子进程的进程ID，为正数；而在子进程中，则返回0。子进程创建成功后，开始执行升级代码，并通过管道与父进程交互，父进程则通过读取子进程传递过来的信息更新UI。
### 7.finish_recovery，重启
上一步完成之后，回到main函数，保存升级过程中的log，清除临时文件，包括command文件（不清除的话，下次重启还会进入recovery模式），最后重启。
```
   // Save logs and clean up before rebooting or shutting down.
    finish_recovery(send_intent);
```
## 附1:cache文件

| 文件 | 输入\输出 | 功能 |
| --- | --- | --- |
| `/cache/recovery/command` | INPUT | MainSystem传递给Recovery的命令行 |
| `/cache/recovery/log` | OUTPUT | Recovery过程的log |
| `/cache/recovery/intent` | OUTPUT | Recovery给MainSystem反馈的信息，比如告诉MainSystem是否升级成功 |
| `/cache/recovery/last_install` | OUTPUT | 上一次的安装包，和TEMPORARY_INSTALL_FILE相关 |
| `/cache/recovery/last_log` | OUTPUT | 保存了recovery的log，一般分析recovery问题时会用到 |
| `/cache/recovery/last_locale` | OUTPUT | 保存了locale信息到cache分区，如果下一次启动recovery的时候没有带-locale参数(例如从bootloader中启动)则会使用这个值 |

## 附2:command命令

| 命令 | 取值 | 含义 |
| ---- | ---- | ---- |
| send_intent | 字符串 | Recovery之后将字符串写到这里，然后写入`/cache/recovery/intent`,也就是升级结果 |
| update_package | path路径 | 安装OTA升级包的路径 |
| wipe_data |    | 擦除user data以及cache,然后重启 |
| wipe_cache |   | 擦除cache，不擦除user data,然后重启 |
| set_encrypted_filesystem | `on\of` | 是否加密文件系统 |
| just_exit |     | 退出和重启 |