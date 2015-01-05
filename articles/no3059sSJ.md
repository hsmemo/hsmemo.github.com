---
layout: default
title: Thread の優先度を変更する処理 (java.lang.Thread.setPriority() の処理)  
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### Thread の優先度を変更する処理 (java.lang.Thread.setPriority() の処理)  

--- 
## 概要(Summary)
基本的には, それぞれの OS におけるスレッドの優先順位変更用のシステムコールを呼び出すだけ.

(なお, java.lang.Thread オブジェクトには priority という private field があり,
 それぞれのスレッドの現在の優先度が格納されている.
 アクセサは java_lang_Thread::priority() と java_lang_Thread::set_priority())

## 備考(Notes)
### java.lang.Thread.setPriority() で指定する優先度の値と OS レベルでの優先度の関係
os::java_to_os_priority という配列で管理されている.
この配列の値は JavaPriority{1,2,3,4,5,6,7,8,9,10}_To_OSPriority コマンドラインオプションで指定することができる.

指定が無ければ以下のデフォルト値が使われる.

* Linux における Java レベルの優先度と OS レベルの優先度の関係


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.cpp))
    int os::java_to_os_priority[MaxPriority + 1] = {
      19,              // 0 Entry should never be used
    
       4,              // 1 MinPriority
       3,              // 2
       2,              // 3
    
       1,              // 4
       0,              // 5 NormPriority
      -1,              // 6
    
      -2,              // 7
      -3,              // 8
      -4,              // 9 NearMaxPriority
    
      -5               // 10 MaxPriority
    };
```

* Solaris における Java レベルの優先度と OS レベルの優先度の関係 (ThreadPriorityPolicy オプションが 0 (デフォルト値) の場合 (?))


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.cpp))
    int os::java_to_os_priority[MaxPriority + 1] = {
      -99999,         // 0 Entry should never be used
    
      0,              // 1 MinPriority
      32,             // 2
      64,             // 3
    
      96,             // 4
      127,            // 5 NormPriority
      127,            // 6
    
      127,            // 7
      127,            // 8
      127,            // 9 NearMaxPriority
    
      127             // 10 MaxPriority
    };
```

* Solaris における Java レベルの優先度と OS レベルの優先度の関係 (ThreadPriorityPolicy オプションが 1 の場合 (?))


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.cpp))
    // Values for ThreadPriorityPolicy == 1
    int prio_policy1[MaxPriority+1] = { -99999, 0, 16, 32, 48, 64,
                                            80, 96, 112, 124, 127 };
```

* Windows における Java レベルの優先度と OS レベルの優先度の関係 (ThreadPriorityPolicy オプションが 0 (デフォルト値) の場合)


```cpp
    ((cite: hotspot/src/os/windows/vm/os_windows.cpp))
    int os::java_to_os_priority[MaxPriority + 1] = {
      THREAD_PRIORITY_IDLE,                         // 0  Entry should never be used
      THREAD_PRIORITY_LOWEST,                       // 1  MinPriority
      THREAD_PRIORITY_LOWEST,                       // 2
      THREAD_PRIORITY_BELOW_NORMAL,                 // 3
      THREAD_PRIORITY_BELOW_NORMAL,                 // 4
      THREAD_PRIORITY_NORMAL,                       // 5  NormPriority
      THREAD_PRIORITY_NORMAL,                       // 6
      THREAD_PRIORITY_ABOVE_NORMAL,                 // 7
      THREAD_PRIORITY_ABOVE_NORMAL,                 // 8
      THREAD_PRIORITY_HIGHEST,                      // 9  NearMaxPriority
      THREAD_PRIORITY_HIGHEST                       // 10 MaxPriority
    };
```

* Windows における Java レベルの優先度と OS レベルの優先度の関係 (ThreadPriorityPolicy オプションが 1 の場合)


```cpp
    ((cite: hotspot/src/os/windows/vm/os_windows.cpp))
    int prio_policy1[MaxPriority + 1] = {
      THREAD_PRIORITY_IDLE,                         // 0  Entry should never be used
      THREAD_PRIORITY_LOWEST,                       // 1  MinPriority
      THREAD_PRIORITY_LOWEST,                       // 2
      THREAD_PRIORITY_BELOW_NORMAL,                 // 3
      THREAD_PRIORITY_BELOW_NORMAL,                 // 4
      THREAD_PRIORITY_NORMAL,                       // 5  NormPriority
      THREAD_PRIORITY_ABOVE_NORMAL,                 // 6
      THREAD_PRIORITY_ABOVE_NORMAL,                 // 7
      THREAD_PRIORITY_HIGHEST,                      // 8
      THREAD_PRIORITY_HIGHEST,                      // 9  NearMaxPriority
      THREAD_PRIORITY_TIME_CRITICAL                 // 10 MaxPriority
    };
```


### Linux における実装について
コメントによると以下の通り
(なお, setpriority() は POSIX の規定ではプロセス単位の優先度制御だが
 Linux ではスレッド単位での制御になっている.
 LinuxThreads はともかく NPTL でさえそうなっている. "pthreads" の man ページも参照).

「Linux には優先度が固定の Real Time thread (SCHED_RR) もあるが, 
(2005年 9月に試した所では) Java のスレッド数の多いプログラムで使うとひどく不安定だったのでお勧めできない.

一応 Linux 用のコードはあるが, 
setpriority() はあるスレッドの優先度だけを変更するものでプロセス全体の優先度を変えるものではないとか, 
JavaThread と OS レベルのスレッドは 1対1対応とか, いろいろ仮定がある.

将来に渡ってそれが維持されるかどうか分からないので, 
Linux 上ではデフォルトでは java.lang.Thread.setPriority() 機能はサポートしない.
使いたい場合は, ThreadPriorityPolicy オプションを 1 にセットし, root 権限を取得して実行する必要がある.」


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.cpp))
    ////////////////////////////////////////////////////////////////////////////////
    // thread priority support
    
    // Note: Normal Linux applications are run with SCHED_OTHER policy. SCHED_OTHER
    // only supports dynamic priority, static priority must be zero. For real-time
    // applications, Linux supports SCHED_RR which allows static priority (1-99).
    // However, for large multi-threaded applications, SCHED_RR is not only slower
    // than SCHED_OTHER, but also very unstable (my volano tests hang hard 4 out
    // of 5 runs - Sep 2005).
    //
    // The following code actually changes the niceness of kernel-thread/LWP. It
    // has an assumption that setpriority() only modifies one kernel-thread/LWP,
    // not the entire user process, and user level threads are 1:1 mapped to kernel
    // threads. It has always been the case, but could change in the future. For
    // this reason, the code should not be used as default (ThreadPriorityPolicy=0).
    // It is only used when ThreadPriorityPolicy=1 and requires root privilege.
```

ただし, 実際の実装は上記のコメントとは少し異なる (バグ?? まぁ root 権限がなければ優先度を上げることは出来ないが...).

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table12387WvU -->
| ThreadPriorityPolicy | description |
|---|---|
| 0 (デフォルト値) の場合 | setpriority() は呼ばれない. |
| 1 にセットした場合 | root 権限がなければ ThreadPriorityPolicy が 0 に戻される (= setpriority() は呼ばれない) (See: prio_init()) |
| 0 でも 1 でもない値にセットした場合 | root 権限がなくても setpriority() が呼ばれる. |
<!-- END RECEIVE ORGTBL table12387WvU -->

<!-- 
#+ORGTBL: SEND table12387WvU orgtbl-to-gfm :no-escape t
| ThreadPriorityPolicy                | description                                                                                                |
|-------------------------------------+------------------------------------------------------------------------------------------------------------|
| 0 (デフォルト値) の場合             | setpriority() は呼ばれない.                                                                                |
| 1 にセットした場合                  | root 権限がなければ ThreadPriorityPolicy が 0 に戻される (= setpriority() は呼ばれない) (See: prio_init()) |
| 0 でも 1 でもない値にセットした場合 | root 権限がなくても setpriority() が呼ばれる.                                                              |
-->

###### 参考(for your information): prio_init() (Linux の場合)
See: [here](no12387xEo.html) for details

### Solaris における LWP 制御について
LWP の優先度は priocntl() で変更できるが, 古い Solaris 8 もサポート対象とするため, 
priocntl_ptr という関数ポインタ経由で priocntl() を呼び出している.

この priocntl_ptr には, 初回の呼び出し時に priocntl_stub() によって正しい関数ポインタがセットされる.
(Solaris 8 以前のバージョンがサポート対象外になれば, この仕組みは排除される予定, とのこと)


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.cpp))
    // Call the version of priocntl suitable for all supported versions
    // of Solaris. We need to call through this wrapper so that we can
    // build on Solaris 9 and run on Solaris 8, 9 and 10.
    //
    // This code should be removed if we ever stop supporting Solaris 8
    // and earlier releases.
    
    static long priocntl_stub(int pcver, idtype_t idtype, id_t id, int cmd, caddr_t arg);
    typedef long (*priocntl_type)(int pcver, idtype_t idtype, id_t id, int cmd, caddr_t arg);
    static priocntl_type priocntl_ptr = priocntl_stub;
```


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
java.lang.Thread.setPriority()
-&gt; JVM_SetThreadPriority() (= java.lang.Thread.setPriority0())
   -&gt; java_lang_Thread::set_priority()
      -&gt; java.lang.Thread オブジェクトの priority フィールドに値をセットするだけ.
   -&gt; Thread::set_priority()
      -&gt; os::set_priority()
         -&gt; os::set_native_priority()
            -&gt; OS によって処理が異なる.
               * Linux の場合
                 (なお, UseThreadPriorities オプションが指定されていない場合や ThreadPriorityPolicy オプションが 0 の場合には, 何もしない)
                 -&gt; setpriority()
               * Solaris の場合
                 (なお, UseThreadPriorities オプションが指定されていない場合は, 何もしない)
                 -&gt; thr_setprio()
                 -&gt; set_lwp_priority()
                    -&gt; priocntl()
               * Windows の場合
                 (なお, UseThreadPriorities オプションが指定されていない場合は, 何もしない)
                 -&gt; SetThreadPriority()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Thread.setPriority()
See: [here](no2114L_p.html) for details
### JVM_SetThreadPriority()
See: [here](no2114YJw.html) for details
#### 備考(Notes)
java.lang.Thread.setPriority0() は JVM_SetThreadPriority() で実装されている.


```cpp
    ((cite: jdk/src/share/native/java/lang/Thread.c))
    static JNINativeMethod methods[] = {
    ...
        {"setPriority0",     "(I)V",       (void *)&JVM_SetThreadPriority},
```

### java_lang_Thread::set_priority()
See: [here](no2114XdF.html) for details
### Thread::set_priority()
See: [here](no2114lT2.html) for details
### os::set_priority()
See: [here](no2114knL.html) for details
### os::set_native_priority()  (Linux の場合)
See: [here](no2114xxR.html) for details
### os::set_native_priority()  (Solaris の場合)
See: [here](no2114YQk.html) for details
### set_lwp_priority()
See: [here](no3059KAO.html) for details
### priocntl_stub()
See: [here](no3059kUa.html) for details
### os::set_native_priority()  (Windows の場合)
See: [here](no2114-7X.html) for details






