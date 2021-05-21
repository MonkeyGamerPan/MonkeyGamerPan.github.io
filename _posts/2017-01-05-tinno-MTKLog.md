---
layout: post
title: android MTKLog相关
featured-img: sleek
mathjax: true
categories: note
---

## MTKLog各部分文件对应log

**crash_log** ：崩溃日志，主要输出 程序崩溃造成的crash log

**events_log**：事件日志，主要输出记录各个activity周期及事件

**kernel_log**：底层驱动，按键，低内存相关log

**sys_log**：系统日志，Exception定位点

**radio_log**：输出通话，网络状态变化

**main_log**：详尽输出每一步的log

</br></br>

## 常见异常分析

#### 1.编译报错：

在build.log 中搜索unfinished 关键字，查找上文能够很快定位报错原因，或者搜索 error: 关键字能够直接定位相关报错文件（注意是搜索error和冒号）

</br>

#### 2.程序崩溃（系统提示***已停止运行）：

（1）启动崩溃：一般情况为第三方预置缺少库文件，或者兼容性问题

（2）应用间交互崩溃：startActivity找不多对应包名或者类名，或者无对应启动Activity的权限

（3）APP内部逻辑空指针异常导致程序崩溃（NullPointerException）

以上三种情况都可在mtklog\mobilelog\APLog_2016_0505_115433\events_log 文件中搜索 crash 关键字快速定位问题点，crash_log中可查看对应问题产生原因：

```
Line 4201: 03-29 11:25:32.894092 939 949 I am_crash:[9337,0,com.bbm,954744388,java.lang.UnsatisfiedLinkError,dalvik.system.PathClassLoader[DexPathList[[zip file"/system/framework/com.google.android.maps.jar", zip file"/data/app/com.bbm-1/base.apk"],nativeLibraryDirectories=[/data/app-lib/com.bbm-1,/data/app/com.bbm-1/base.apk!/lib/armeabi-v7a, /vendor/lib, /system/lib]]]couldn't find "libgnustl_shared.so",Runtime.java,367]
Line 4202: 03-29 11:25:32.910190 939 949 Iam_finish_activity: [0,198242511,14,com.bbm/.ui.activities.StartupActivity,force-cras
```

</br>

#### 3.程序闪退：

（1）外部原因：物理内存不足，被kill，events_log中搜索 low_memory 关键字，以确定低内存杀死程序，kernal_log 中有存在对应时间点被 low memory kill.

（2）内部原因：main_log/sys_log 搜索Exception 或者 died关键字定位对应包名，进而定位问题

</br>

#### 4.ANR问题：

出现ANR应当提供traces.txt文件，直接在文件中搜索 cmd 关键字，定位问题点。锁定三个方向：memoryleak（是否为低内存），CPU block（CPU使用率过高）、iowait（IO流使用过于频繁）

（1）memoryleak：首先根据Android log搜索低内存相关 low_memory 关键字，以确定是否存在低内存现象

（2）CPU block：搜索对应包出现ANR前后 TOTAL 关键字前的百分比，若百分比接近100% 说明CPU饥饿导致了ANR：

（3）iowait：搜索iowait 关键字查看出现ANR前的百分比，若百分比过高，说明I/O流使用过于频繁导致ANR，此项需修改相关数据库的加载流程。

</br>

#### 5.关机:

关机一般就几种情况 低电，高温，掉电。

</br>

#### 6.系统重启:

查看sys_log.log中是否存在触发watch dog的情况，如果存在则需要根据pid到data/anr/watchdog文件夹下查看对应的记录文件并分析原因.

</br>
#### 7.Watchdog相关Log分析

#####导致SWT的原因
   线程死锁
   Binder的server端卡住
   Native方法执行时间过长
   SurfaceFlinger卡住
   Dump时间过长
   Zygote fork进程时卡住
   Binder used up
</br>
</br>

https://blog.csdn.net/ch853199769/article/details/88244689#t6




