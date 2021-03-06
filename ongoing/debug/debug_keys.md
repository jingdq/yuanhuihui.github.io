## 时间问题

1. up time: 00:30:16, idle time: 00:29:30, sleep time: 00:02:41 (重点看up time)


## 一. Java Crash

1. system log

system_server crash

    *** FATAL EXCEPTION IN SYSTEM PROCESS [线程名]

app crash

    FATAL EXCEPTION: [线程名]
    Process: [进程名], PID: [进程id]

技巧1: 搜索关键词`FATAL EXCEPTION`来查看所有的Java crash问题,


2. event Log

技巧2: Event log中搜索"am_crash", am_crash格式:

    //[pid, UserId, 进程名, flags, Exception类, Exception内容, 抛异常文件名, 抛异常行号]
    am_crash: [16208,0,com.android.camera,952745541,java.lang.RuntimeException,getParameters failed (empty parameters),CameraManager.java,262]

定义:

    EventLog.writeEvent(EventLogTags.AM_CRASH,
        Binder.getCallingPid(),
        UserHandle.getUserId(Binder.getCallingUid()),
        processName,
        r == null ? -1 : r.info.flags,
        crashInfo.exceptionClassName,
        crashInfo.exceptionMessage,
        crashInfo.throwFileName,
        crashInfo.throwLineNumber);

3. dropbox

位于`/data/system/dropbox`

    system_server_crash
    system_app_crash
    data_app_crash

主要输出输出内容:

- Process,flags, package等头信息；
- stacktrace栈信息;


**小结:system server crash关键词**

    FATAL EXCEPTION IN SYSTEM PROCESS
    system_server_crash


## 二. Watchdog

默认timeout=60s,调试时才为10s方便找出潜在的ANR问题

输出内容项:

    遍历输出阻塞线程的栈信息
        AMS.dumpStackTraces
            kill -3
            backtrace.dump_backtrace()
        dumpKernelStackTraces，输出kernel栈信息
        doSysRq('l');
        生成文件到dropBox,文件名system_server_watchdog前缀
    杀死system_server

输出文件:


**小结:system server watchdog关键词**

    WATCHDOG KILLING SYSTEM PROCESS
    system_server_watchdog


### 这里有几个问题, AMS.dumpStackTraces, dumpKernelStackTraces，doSysRq???

## 三. Native Crash

native程序(C/C++)出现异常时，kernel会发送相应的signal, 当进程捕获致命的signal，将action=DEBUGGER_ACTION_CRASH的消息发送给debuggerd服务端,阻塞等待回应消息.

输出内容项:

    pid: %d, tid: %d, name: [线程名]  >>> [进程名] <<<
    signal相关信息，包含fault address
    寄存器状态
    backtrace
    stack
    memory near
    code around
    memory map

输出文件:

`/data/tombstones/tombstone_XX`

**小结:system server Native crash关键词**

    >>> system_server <<< (位于tombstone)

## 四. 总结

### 4.1 system进程

对于system_server异常需要关注的关键词:

**Java Crash:**

    FATAL EXCEPTION IN SYSTEM PROCESS (位于system log)
    system_server_crash (位于dropbox)

**watchdog:**

    WATCHDOG KILLING SYSTEM PROCESS (位于system log)
    system_server_watchdog  (位于dropbox)

**Native Crash:**

    >>> system_server <<< (位于tombstone)

### 4.2 普通进程

**Java Crash:**

    FATAL EXCEPTION (位于system log)
    am_crash (位于event log)
    system_app_crash (位于dropbox)
    data_app_crash (位于dropbox)

**Native Crash:**

    >>>   (位于tombstone)




## 其他


echo "parsing ZYGOTE WAS GONE..."
grep -rn "Exit zygote" ./ --color

echo "parsing ZYGOTE WAS GONE BY EVENT LOG..."
grep -rnC 1 "boot_progress_start" ./ --color



echo "parsing SYSTEM_SERVER_HANG ..."

grep -rn "traces_SystemServer_WDT" ./ --color
grep -rn "am_watchdog" ./ --color

grep -rn "watchdog since start" ./ --color
grep -rn "Blocked in" ./ --color

echo "**********parsing SysRq*************..."
grep -rn "Show Blocked State" ./ --color


echo "parse SYSTEM_RESTART..."
grep -rn "SYSTEM_RESTART" ./ --color

echo "parse SYSTEM_BOOT..."
grep -rn "SYSTEM_BOOT" ./ --color

echo "parse KERNEL ISSUE.."
grep -rn "kpanic" ./ --color
grep -rn "Kernel BUG" ./ --color
grep -rn "Internal error" ./ --color
grep -rn "Kernel panic" ./ --color
grep -rn "Watchdog bark" ./ --color
grep -rn "watchdog bite" ./ --color
grep -rn "kernel reboot     :" ./ --color

echo "parse MODEM RESET.."
grep -rn "Modem Restart" ./ --color
grep -rn "Fatal error" ./ --color

echo "parse PUREASON.."
grep -rn "PuReason:" ./ --color
grep -rn "Powerup reason" ./ --color

echo "parse BOOTFAILTURE..."
grep -rn "BOOT FAILURE" ./ --color


echo "parse ShenYinMoShi"
grep -rn "mfrozenAppFunCtrlFlg" ./ --color
grep -rn "STATE is" ./ --color
#grep -rnE -A10 "#+dump FrozenAppController#+" ./ --color

echo "parse dropbox entries..."
grep -rn "Drop box contents" ./ --color

echo "parse ANR ..."
grep -rn "am_anr" ./ --color
grep -rn "system_app_anr" ./ --color
grep -rn " VM TRACES" ./ --color

echo "parse system app native crash"
grep -rn -A3 "system_app_native_crash" ./ --color

#echo "parse system tombstone"
#grep -rn -A3 "SYSTEM_TOMBSTONE" ./ --color
#echo "parse SYSTEM_APP_WTF"


## else

force acquire UnstableProvider
acquire killed Provider
is crashing

cat bugreport_1472203656562.log | egrep "force acquire UnstableProvider|acquire killed Provider|is crashing|depends on"

## D状态

DEBUG : timed out waiting for stop signal: tid=7601
DEBUG : detach failed: tid 7601, No such process
debuggerd committing suicide to free the zombie
