---
title: 定制开关保存LOGCAT和KMSG日志输出到文件
date: 2019-02-20 15:49:09
categories: Android Framework
tags: [Android]
toc: true
description: I'm trying not to pay too much attention to the rankings because calculations can distract you.
---
在 "设置" 应用中的开发者选项添加一个开关 保存Logcat和KMSG日志

# 添加te文件

由于SELinux的原因,需要在sepolicy下添加catlot.te

```
type catlog, domain;
type catlog_exec, exec_type, file_type;

allow init catlog_exec:file { execute getattr read open };
allow init catlog:process { transition };
allow init catlog:process { rlimitinh siginh noatsecure };
allow catlog kernel:system { syslog_mod };
allow catlog catlog:capability { dac_override sys_nice };
allow catlog catlog:capability2 { syslog };
allow catlog catlog_exec:file { execute entrypoint read open };
allow catlog shell_exec:file { getattr read };
allow catlog rootfs:lnk_file { getattr };
allow catlog proc:file { write open read };
allow catlog tmpfs:lnk_file { read };
allow catlog storage_file:dir { search };
allow catlog storage_file:lnk_file { read };
allow catlog mnt_user_file:dir { search };
allow catlog mnt_user_file:lnk_file { read };
allow catlog fuse:dir { search getattr create write read open add_name rename remove_name };
allow catlog fuse:file { getattr create write open rename append };
allow catlog toolbox_exec:file { execute read open getattr execute_no_trans };
allow catlog logdr_socket:sock_file { write };
allow catlog logd:unix_stream_socket { connectto };
allow catlog logcat_exec:file { execute read open execute_no_trans getattr };
```

# 添加`cat_log.sh`脚本

```
#!/system/bin/sh

# 在init.rc下加入如下语句
# service  catlog /system/bin/busybox  sh  /system/bin/cat_log.sh
#     disabled
#     oneshot
#
# on property:sys.boot_completed=1
#   start catlog
#

function enable_log()
{
       LOG_FILE="/data/tool.log"

       exec 2>> $LOG_FILE
       exec 1>> $LOG_FILE
       echo "----------------------------------"
       echo "para: $*"
}

enable_log $* ; set -x


echo 1 > /proc/sys/kernel/panic

#命名文件夹名字e5_log,也可以自己更改
ENABLE_LOG_FILE="/mnt/sdcard/.enable_logsave"
LOG_DIR="/mnt/sdcard/LOGSAVE"
LAST_LOG_DIR="/mnt/sdcard/LOGSAVE/last"
SAVE_LOG_COUNT=5   # 保存上5次的log，值最小为1;例为5,则last.1为最后一次重启前的log；last.5为最老的log

#echo “save last_time  ${SAVE_LOG_COUNT} log”


#if [ ! -f "$ENABLE_LOG_FILE" ];then                     
#   echo "disable logsave"                    
#   exit                                                 

#fi 

#echo "enable logsave"

if [ ! -d "$LOG_DIR" ];then
   mkdir $LOG_DIR
fi

if [  -d "$LAST_LOG_DIR.$SAVE_LOG_COUNT" ];then
   rm -r "$LAST_LOG_DIR.$SAVE_LOG_COUNT"
fi


#for((i= ${SAVE_LOG_COUNT}-1; $i >= 1 ;i--));do
#for i in  $(seq  `expr $SAVE_LOG_COUNT - 1`  -1 1)
i=$((SAVE_LOG_COUNT -1))
while [ $i -ge 1 ]
do 
        if [  -d "$LAST_LOG_DIR.$i" ];then
               #echo "$LAST_LOG_DIR.$i is exists "
               if [ "`ls -a $LAST_LOG_DIR.$i`" = "" ]; then
                       echo "$LAST_LOG_DIR.$i is indeed empty"
               else
                       echo "$LAST_LOG_DIR.$i is not empty"
                       #j=`expr $i + 1`
                       j=$(($i+1)) 
                       mv  "$LAST_LOG_DIR.$i"  "$LAST_LOG_DIR.$j"
               fi


       #else
               #echo "$LAST_LOG_DIR.$i isnot exists"

        fi
        i=$(($i-1))

done

#创建上一次日志保存目录
mkdir $LAST_LOG_DIR."1"

#保存上次开机之后的log
mv $LOG_DIR/*.log $LAST_LOG_DIR."1"
mv $LOG_DIR/*.log* $LAST_LOG_DIR."1"


DATE=$(date +%Y%m%d%H%M)

cat /sys/fs/pstore/console-ramoops"-0" > $LOG_DIR/"$DATE"_panic_kmsg.log

echo "------start kmsg log------"
cat /proc/kmsg > $LOG_DIR/"$DATE"_kmsg.log &

echo "------start logcat log------"
logcat -v time -n 1 -f $LOG_DIR/"$DATE"_logcat.log -r10240 
```

# 拷贝脚本到`system/bin`

在mk文件添加

```
PRODUCT_COPY_FILES +=$(CUR_PATH)/cat_log.sh:system/bin/cat_log.sh
```

# `property_service`里赋予权限

在property_service.cpp 的检查权限check_mac_perms函数中添加

```
 if(strcmp("persist.sys.cat_log",name) == 0)
    {
        return 1;
    }
```

# `init.*.rc`里声明服务

```
#catlog
service catlog /system/bin/cat_log.sh
    seclabel u:r:catlog:s0
    disabled
    oneshot

on property:persist.sys.cat_log=1
     start catlog

on property:persist.sys.cat_log=0
     stop catlog
```

>其中的seclabel就是SELinux要用到的标志

# 开发者选项中添加一个SwitchPreference组件

```
<SwitchPreference
    android:key="logcat_enable"
    android:title="@string/logcat_enable"
    android:summary="@string/logcat_summary"
    android:fragment="com.android.tv.settings.system.development.AdbDialog" />
```

# 添加开关控制逻辑

在DevelopmentFragment.java的onPreferenceTreeClick里添加相关逻辑

```
if(preference == mLogcatPreference){
            if (mLogcatPreference.isChecked()){
                writeLogcatEnableOptions(1);
                setLogcatEnable(getPreferenceManager().getContext(), 1);
            } else {
                writeLogcatEnableOptions(0);
                setLogcatEnable(getPreferenceManager().getContext(), 0);
            }
        }
```

# 开机时初始化状态

```
String action = intent.getAction();
            if (action.equals(Intent.ACTION_BOOT_COMPLETED)) {
                SharedPreferences sharedPreferences = context
                        .getSharedPreferences("TvSetting", Context.MODE_PRIVATE);
                boolean enable_log_save = sharedPreferences.getBoolean(
                        "enable_log_save", false);
                String persist_sys_cat_log = SystemProperties.get("persist.sys.cat_log");
                Log.i("BootReceiver",
                        "action==ACTION_BOOT_COMPLETED,BootReceiver is start");
                if (persist_sys_cat_log.equals("1")) {
                    SystemProperties.set("persist.sys.cat_log", "1");
                }
            }
```

编译运行，打开开关，可以看到LOG保存到了sdcard/LOGSAVE目录下面。