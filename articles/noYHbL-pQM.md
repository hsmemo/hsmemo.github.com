---
layout: default
title: Thread の開始処理の枠組み ： スレッドを生成する側の処理
---
[Up](no3059-9C.html) [Top](../index.html)

#### Thread の開始処理の枠組み ： スレッドを生成する側の処理

--- 
## 概要(Summary)
生成処理の流れは以下のようになる.

1. Thread オブジェクトの生成処理

   スレッドは Thread クラスのサブクラスとして表現されている.

   まず適切な Thread サブクラスのオブジェクトが生成される.

2. スレッドを生成する処理

   os::create_thread() を呼び出すと, 実際にスレッドの生成が行われる.

3. 生成したスレッドの実行を開始させる処理

   生成直後のスレッドは実行が停止されている.

   os::start_thread() を呼ぶことで, 実際に実行が開始される.


## 処理の流れ (概要)(Execution Flows : Summary)
### スレッドを生成する処理
<div class="flow-abst"><pre>
-&gt; (Thread クラスの種々のサブクラスのコンストラクタ)
   -&gt; Thread::Thread()

-&gt; os::create_thread()
   -&gt; OS によって処理が異なる.
      * Linux の場合
        -&gt; OSThread::OSThread()
           -&gt; OSThread::pd_initialize()
        -&gt; pthread_attr_setdetachstate()
        -&gt; pthread_attr_setstacksize()   (&lt;= 必要があれば呼び出す)
        -&gt; pthread_attr_setguardsize()
        -&gt; pthread_create()              (&lt;= なお, エントリポイントとしては java_start() 関数が指定されている)
        -&gt; Monitor::wait()
           (生成したスレッドと同期を取る処理. java_start() 内から Monitor::notify_all() されるまで待機)
      * Solaris の場合
        -&gt; OSThread::OSThread()
           -&gt; OSThread::pd_initialize()
        -&gt; thr_setconcurrency()          (&lt;= 必要があれば呼び出す)
        -&gt; thr_create()                  (&lt;= なお, エントリポイントとしては java_start() 関数が指定されている)
        -&gt; thr_setprio()                 (&lt;= 必要があれば呼び出す)
      * Windows の場合
        -&gt; OSThread::OSThread()
           -&gt; OSThread::pd_initialize()
        -&gt; CreateEvent()
        -&gt; _beginthredex()               (&lt;= なお, エントリポイントとしては java_start() 関数が指定されている)
</pre></div>

### 生成したスレッドの実行を開始させる処理
<div class="flow-abst"><pre>
-&gt; os::start_thread()
   -&gt; OSThread::set_state()
   -&gt; os::pd_start_thread()
      -&gt; OS によって処理が異なる.
         * Linux の場合
           -&gt; Monitor::notify()
              (生成したスレッドと同期を取る処理. java_start() 内で Monitor::wait() しているスレッドを起床させる)
         * Solaris の場合
           -&gt; thr_continue()
         * Windows の場合
           -&gt; ResumeThread()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### Thread::Thread()
See: [here](no2114wpW.html) for details
### os::create_thread()  (Linux の場合)
See: [here](no2114xc1.html) for details
### os::Linux::default_guard_size()  (Linux x86 の場合)
See: [here](no211496Q.html) for details
### os::Linux::default_guard_size()  (Linux sparc の場合)
See: [here](no2114wwK.html) for details
### JavaThread::stack_size_at_create()
See: [here](no30595Vb.html) for details
### os::create_thread()  (Solaris の場合)
See: [here](no2114jmE.html) for details
### OSThread::set_vm_created()
See: [here](no3059-om.html) for details
### os::create_thread()  (Windows の場合)
See: [here](no3059sLV.html) for details
### OSThread::OSThread()
See: [here](no3059Ggh.html) for details
### OSThread::pd_initialize()  (Linux の場合)
See: [here](no3059Tqn.html) for details
### OSThread::pd_initialize()  (Solaris の場合)
See: [here](no3059g0t.html) for details
### OSThread::pd_initialize()  (Windows の場合)
See: [here](no3059t-z.html) for details

### os::start_thread()
See: [here](no3059Ha0.html) for details
### OSThread::set_state()
See: [here](no30595jD.html) for details
### os::pd_start_thread() (Linux の場合)
See: [here](no3059GuJ.html) for details
### os::pd_start_thread() (Solaris の場合)
See: [here](no3059T4P.html) for details
### os::pd_start_thread() (Windows の場合)
See: [here](no3059gCW.html) for details






