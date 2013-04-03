---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： スレッド毎の CPU 使用時間の取得処理 (java.lang.management.ThreadMXBean.get*Time() の処理)
---
[Up](noMz-1isvk.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： スレッド毎の CPU 使用時間の取得処理 (java.lang.management.ThreadMXBean.get*Time() の処理)

--- 
## 概要(Summary)
これらは JSR-174 標準のスレッドアカウンティング機能, および HotSpot 独自のアカウンティング機能.
このうち HotSpot 独自のメソッド (= com.sun.management.ThreadMXBean で追加されているメソッド) は以下の 2つ.

  * com.sun.management.ThreadMXBean.getThreadCpuTime(long[] ids)
  * com.sun.management.ThreadMXBean.getThreadUserTime(long[] ids)

ただし, どのメソッドも最終的には各 OS が提供しているアカウンティング機能を呼び出すだけ.
より具体的に言うと, 各 OS の以下の機能を利用する
(See: os::current_thread_cpu_time(), os::thread_cpu_time()).

  * Solaris
      * user時間のみを取得する場合:
        gethrvtime() で取得する
      * user時間+sys時間 を取得する場合:
        proc ファイルシステムから取得する

  * Linux
      * user時間のみを取得する場合:
        proc ファイルシステムから取得する
      * user時間+sys時間 を取得する場合:
        利用可能なら clock_gettime() で取得する (駄目なら proc ファイルシステムから取得する).

  * Windows
      * NT 系のカーネルの場合:
        GetThreadTimes() で取得する.
      * そうではない場合:
        timeGetTime() で取得する.


```
    ((cite: hotspot/src/share/vm/runtime/os.hpp))
      // JVMTI & JVM monitoring and management support
      // The thread_cpu_time() and current_thread_cpu_time() are only
      // supported if is_thread_cpu_time_supported() returns true.
      // They are not supported on Solaris T1.
    
      // Thread CPU Time - return the fast estimate on a platform
      // On Solaris - call gethrvtime (fast) - user time only
      // On Linux   - fast clock_gettime where available - user+sys
      //            - otherwise: very slow /proc fs - user+sys
      // On Windows - GetThreadTimes - user+sys
      static jlong current_thread_cpu_time();
      static jlong thread_cpu_time(Thread* t);
    
      // Thread CPU Time with user_sys_cpu_time parameter.
      //
      // If user_sys_cpu_time is true, user+sys time is returned.
      // Otherwise, only user time is returned
      static jlong current_thread_cpu_time(bool user_sys_cpu_time);
      static jlong thread_cpu_time(Thread* t, bool user_sys_cpu_time);
```

## 備考(Notes)
Solaris 上での実装について:


```
    ((cite: hotspot/src/os/solaris/vm/os_solaris.cpp))
    // JVMTI & JVM monitoring and management support
    // The thread_cpu_time() and current_thread_cpu_time() are only
    // supported if is_thread_cpu_time_supported() returns true.
    // They are not supported on Solaris T1.
    
    // current_thread_cpu_time(bool) and thread_cpu_time(Thread*, bool)
    // are used by JVM M&M and JVMTI to get user+sys or user CPU time
    // of a thread.
    //
    // current_thread_cpu_time() and thread_cpu_time(Thread *)
    // returns the fast estimate available on the platform.
```


## 処理の流れ (概要)(Execution Flows : Summary)
```
sun.management.ThreadImpl.getCurrentThreadCpuTime()
-> sun.management.ThreadImpl.getThreadTotalCpuTime0()
   -> Java_sun_management_ThreadImpl_getThreadTotalCpuTime0()
      -> jmm_GetThreadCpuTimeWithKind()
         以下のどちらかを呼び出す.
         -> os::current_thread_cpu_time()
         -> os::thread_cpu_time()
```

```
sun.management.ThreadImpl.getThreadCpuTime(long id)
-> sun.management.ThreadImpl.getThreadCpuTime(long[] ids)
   以下のどちらかを呼び出す.
   -> sun.management.ThreadImpl.getThreadTotalCpuTime0()
      -> (同上)
   -> sun.management.ThreadImpl.getThreadTotalCpuTime1()
      -> Java_sun_management_ThreadImpl_getThreadTotalCpuTime1()
         -> jmm_GetThreadCpuTimesWithKind()
            -> os::thread_cpu_time()
```

```
sun.management.ThreadImpl.getThreadCpuTime(long[] ids)
-> (同上)
```

```
sun.management.ThreadImpl.getCurrentThreadUserTime()
-> sun.management.ThreadImpl.getThreadUserCpuTime0()
   -> Java_sun_management_ThreadImpl_getThreadUserCpuTime0()
      -> jmm_GetThreadCpuTimeWithKind()
         -> (同上)
```

```
sun.management.ThreadImpl.getThreadUserTime(long id)
-> sun.management.ThreadImpl.getThreadUserTime(long[] ids)
   以下のどちらかを呼び出す.
   -> sun.management.ThreadImpl.getThreadUserCpuTime0()
      -> (同上)
   -> sun.management.ThreadImpl.getThreadUserCpuTime1()
      -> Java_sun_management_ThreadImpl_getThreadUserCpuTime1()
         -> jmm_GetThreadCpuTimesWithKind()
            -> (同上)
```

```
sun.management.ThreadImpl.getThreadUserTime(long[] ids)
-> (同上)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### sun.management.ThreadImpl.getCurrentThreadCpuTime()
See: [here](no2114rbV.html) for details
### Java_sun_management_ThreadImpl_getThreadTotalCpuTime0()
See: [here](no21144lb.html) for details
### jmm_GetThreadCpuTimeWithKind()
See: [here](no2114fLi.html) for details

### sun.management.ThreadImpl.getThreadCpuTime(long id)
See: [here](no2114sO0.html) for details
### sun.management.ThreadImpl.getThreadCpuTime(long[] ids)
See: [here](no2114eYD.html) for details
### Java_sun_management_ThreadImpl_getThreadTotalCpuTime1()
See: [here](no2114Fwh.html) for details
### jmm_GetThreadCpuTimesWithKind()
See: [here](no2114sVo.html) for details

### sun.management.ThreadImpl.getCurrentThreadUserTime()
See: [here](no2114riJ.html) for details
### Java_sun_management_ThreadImpl_getThreadUserCpuTime0()
See: [here](no2114S6n.html) for details
### sun.management.ThreadImpl.getThreadUserTime(long id)
See: [here](no21144sP.html) for details
### sun.management.ThreadImpl.getThreadUserTime(long[] ids)
See: [here](no2114F3V.html) for details
### Java_sun_management_ThreadImpl_getThreadUserCpuTime1()
See: [here](no2114fEu.html) for details

### os::current_thread_cpu_time(bool user_sys_cpu_time) (Solaris の場合)
See: [here](no2114F-J.html) for details
### os::thread_cpu_time() (Solaris の場合)
See: [here](no2114scc.html) for details
### os::current_thread_cpu_time() (Solaris の場合)
See: [here](no2114fSW.html) for details

### os::current_thread_cpu_time() (Linux の場合)
See: [here](no21145fu.html) for details
### os::Linux::fast_thread_cpu_time() (Linux の場合)
See: [here](no21144zD.html) for details
### slow_thread_cpu_time() (Linux の場合)
See: [here](no2114Gq0.html) for details
### os::thread_cpu_time() (Linux の場合)
See: [here](no2114T7u.html) for details

### os::current_thread_cpu_time() (Windows の場合)
See: [here](no21145mi.html) for details
### os::thread_cpu_time() (Windows の場合)
See: [here](no2114Gxo.html) for details






