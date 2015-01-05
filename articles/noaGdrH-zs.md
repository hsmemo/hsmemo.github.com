---
layout: default
title: Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Linux の場合
---
[Up](no3059-9C.html) [Top](../index.html)

#### Thread の開始処理の枠組み ： 生成されたスレッド側での動き ： Linux の場合

--- 
## 概要(Summary)
生成されたスレッド側の処理の流れは以下のようになる.

1. 実行が開始されると java_start() 関数から処理が始まる.

2. java_start() からは最終的に Thread::run() が呼び出される.
   
   (なお, java_start() は各 OS 毎に実装されている.
   ただし, どの場合でも最終的には Thread::run() に行き着く)

3. この Thread::run() は virtual function であり, 
   実際には Thread の各サブクラスでオーバーライドされた run() メソッドが実行される.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
java_start()
-&gt; (1) TLS の設定
       -&gt; ThreadLocalStorage::set_thread()

   (1) スレッドの作成に失敗した場合は, ここで OSThread::startThread_lock() に対して Monitor::notify_all() して終了.
       (生成したスレッドと同期を取る処理)

   (1) NUMA 関係の設定
       -&gt; os::numa_get_group_id()
       -&gt; Thread::set_lgrp_id()

   (1) シグナルマスクの設定
       -&gt; os::Linux::hotspot_sigmask()
          -&gt; (See: <a href="noNmlmYDJk.html">here</a> for details)

   (1) FPU 関係の初期化
       -&gt; os::Linux::init_thread_fpu_state()

   (1) 生成元のスレッドと同期を取る
       -&gt; Monitor::notify_all()
          (OSThread::startThread_lock() に対して notify_all())
       -&gt; Monitor::wait()
          (OSThread::startThread_lock() に対して wait)

   (1) 実際にこのスレッドのメイン処理を実行
       -&gt; Thread::run()
          -&gt; (Thread の各サブクラスでオーバーライドされた run() メソッドが呼び出される)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### java_start() (Linux の場合)
See: [here](no3059tMc.html) for details
### ThreadLocalStorage::set_thread()
See: [here](no30596Wi.html) for details
### ThreadLocalStorage::pd_set_thread() (Linux x86 の場合)
See: [here](no17119wf2.html) for details
### os::thread_local_storage_at_put() (Linux の場合)
See: [here](no3059Uru.html) for details
### os::Linux::gettid()
See: [here](no3059T_D.html) for details
### OSThread::set_thread_id()
See: [here](no3059h10.html) for details
### _thread_safety_check()
See: [here](no3059Hoc.html) for details
### os::Linux::is_LinuxThreads()
(#Under Construction)

### os::Linux::is_floating_stack()
See: [here](no3059Uyi.html) for details
### os::current_stack_base()  (Linux x86 の場合)
See: [here](no3059Iiv.html) for details
### os::current_stack_base()  (Linux sparc の場合)
(#Under Construction)

### os::current_stack_base()  (Linux zero の場合)
(#Under Construction)

### current_stack_region()  (Linux x86 の場合)
See: [here](no3059Vs1.html) for details
### current_stack_region()  (Linux sparc の場合)
(#Under Construction)

### current_stack_region()  (Linux zero の場合)
(#Under Construction)

### os::Linux::is_initial_thread()
See: [here](no3059H2E.html) for details
### os::Linux::initial_thread_stack_bottom()
See: [here](no3059UAL.html) for details
### os::Linux::initial_thread_stack_size()
See: [here](no3059hKR.html) for details
### os::current_stack_size()  (Linux x86 の場合)
See: [here](no3059uUX.html) for details
### os::current_stack_size()  (Linux sparc の場合)
(#Under Construction)

### os::current_stack_size()  (Linux zero の場合)
(#Under Construction)

### highest_vm_reserved_address()
(#Under Construction)

### os::numa_get_group_id()  (Linux の場合)
(#Under Construction)

### Thread::set_lgrp_id()
See: [here](no3059-hy.html) for details
### os::Linux::init_thread_fpu_state() (x86 の場合)
See: [here](no3059h8o.html) for details
### os::Linux::set_fpu_control_word() (x86 の場合)
See: [here](no3059uGv.html) for details
### os::Linux::init_thread_fpu_state() (sparc の場合)
See: [here](no30597Q1.html) for details






