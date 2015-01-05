---
layout: default
title: 生成された java.lang.Thread (= JavaThread) 側での処理 (2) ： スレッドの起動処理(JavaThread 独自の部分)  
---
[Up](no_adcwNt_.html) [Top](../index.html)

#### 生成された java.lang.Thread (= JavaThread) 側での処理 (2) ： スレッドの起動処理(JavaThread 独自の部分)  

--- 
## 概要(Summary)
スレッドの起動処理により Thread::run() が呼び出される (See: [here](no3059-9C.html) for details).
JavaThread は Thread::run() をオーバーライドしているので, 
実際に呼び出されるのは JavaThraed::run() になる.

JavaThread::run() からは, 
最終的に JavaThread オブジェクト生成時に指定された thread_entry() 関数が呼び出される.
この thread_entry() 内でメインの処理を行う java.lang.Thread.run() が呼び出される.

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
-&gt; JavaThread::run()
   -&gt; (1) TLAB(ThreadLocalAllocBuffer) の初期化を行う.
          -&gt; JavaThread::initialize_tlab()
             -&gt; ThreadLocalAllocBuffer::initialize()

      (1) スタック領域関連のフィールドを初期化する.
          -&gt; JavaThread::record_base_of_stack_pointer()

      (1) スタック領域関連のフィールドを初期化する.
          -&gt; Thread::record_stack_base_and_size()

      (1) 現在実行中のネイティブスレッドに, この JavaThread オブジェクトを対応付ける.
          -&gt; Thread::initialize_thread_local_storage()

      (1) カレントスレッドのスタック上に guard page を設定.
          -&gt; JavaThread::create_stack_guard_pages()

      (1) プラットフォーム固有のフィールドを(もしそんなものがあれば)初期化.
          -&gt; JavaThread::cache_global_variables()

      (1) カレントスレッドの JavaThreadState を _thread_new から _thread_in_vm に変更.
          -&gt; ThreadStateTransition::transition_and_fence()

      (1) カレントスレッドの JNI ローカル参照フレームを作成.
          -&gt; JNIHandleBlock::allocate_block()
          -&gt; Thread::set_active_handles()

      (1) カレントスレッドのメイン処理(及びメイン処理終了後の後片付け処理)を行う.
          -&gt; JavaThread::thread_main_inner()
             -&gt; (1) カレントスレッドのメイン処理を実行する
                    -&gt; JavaThread::entry_point()()
                       (JavaThread::entry_point() が thread_entry へのポインタを返し, それが呼び出される)
                        これにより, コンストラクタ引数で渡されていた thread_entry() が呼び出される.
                    -&gt; thread_entry()
                       -&gt; JavaCalls::call_virtual()
                          -&gt; (See: <a href="no3059iJu.html">here</a> for details)
                             -&gt; java.lang.Thread.run()
                                (ここで実際の処理が行われる.
                                 Thread のサブクラスを作った場合は, そちらでオーバライドした run() メソッドが呼ばれる.
                                 そうでなければ, java.lang.Thread.run() の中で,
                                 登録した Runnable オブジェクトの run() メソッドが呼ばれる)

                (1) メイン処理終了後の後片付けを行う
                    -&gt; JavaThread::exit()
                       -&gt; (See: <a href="no2935w3j.html">here</a> for details)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### JavaThread::run()
See: [here](no3059taE.html) for details
### JavaThread::initialize_tlab()
See: [here](no30596kK.html) for details
### ThreadLocalAllocBuffer::initialize()
See: [here](no3059HvQ.html) for details
### ThreadLocalAllocBuffer::initialize(HeapWord* start, HeapWord* top, HeapWord* end)
See: [here](no3059U5W.html) for details
### ThreadLocalAllocBuffer::invariants()
See: [here](no3059hDd.html) for details
### JavaThread::record_base_of_stack_pointer()  (Linux x86 の場合)
See: [here](no3059uNj.html) for details
### JavaThread::record_base_of_stack_pointer()  (Linux sparc の場合)
(#Under Construction)

### JavaThread::record_base_of_stack_pointer()  (Linux zero の場合)
(#Under Construction)

### JavaThread::record_base_of_stack_pointer()  (Solaris sparc の場合)
(#Under Construction)

### JavaThread::record_base_of_stack_pointer()  (Solaris x86 の場合)
(#Under Construction)

### JavaThread::record_base_of_stack_pointer()  (Windows x86 の場合)
(#Under Construction)

### Thread::record_stack_base_and_size()
See: [here](no30597Xp.html) for details
### os::current_stack_base()  (Linux x86 の場合)
See: [here](no3059Iiv.html) for details
### current_stack_region()  (Linux x86 の場合)
See: [here](no3059Vs1.html) for details
### os::Linux::is_initial_thread()
See: [here](no3059H2E.html) for details
### os::Linux::initial_thread_stack_bottom()
See: [here](no3059UAL.html) for details
### os::Linux::initial_thread_stack_size()
See: [here](no3059hKR.html) for details
### os::current_stack_size()  (Linux x86 の場合)
See: [here](no3059uUX.html) for details
### Thread::initialize_thread_local_storage()
See: [here](no17119iiR.html) for details
### ThreadLocalStorage::set_thread()
See: [here](no30596Wi.html) for details
### JavaThread::create_stack_guard_pages()
See: [here](no3059Ipj.html) for details
### os::uses_stack_guard_pages()  (Linux の場合)
See: [here](no3059kGy.html) for details
### os::uses_stack_guard_pages()  (Solaris の場合)
See: [here](no3059WQB.html) for details
### os::uses_stack_guard_pages()  (Windows の場合)
See: [here](no3059jaH.html) for details
### os::allocate_stack_guard_pages()  (Linux の場合)
See: [here](no30599gr.html) for details
### os::allocate_stack_guard_pages()  (Solaris の場合)
See: [here](no305980A.html) for details
### os::allocate_stack_guard_pages()  (Windows の場合)
See: [here](no3059Krx.html) for details
### os::create_stack_guard_pages()  (Linux の場合)
See: [here](no30597ed.html) for details
### os::create_stack_guard_pages()  (Solaris の場合)
See: [here](no3059J_G.html) for details
### os::create_stack_guard_pages()  (Windows の場合)
See: [here](no3059WJN.html) for details
### os::guard_memory()  (Linux の場合)
See: [here](no3059jTT.html) for details
### linux_mprotect()
See: [here](no3059Kyl.html) for details
### os::guard_memory()  (Solaris の場合)
See: [here](no3059wdZ.html) for details
### solaris_mprotect()
See: [here](no3059X8r.html) for details
### os::guard_memory()  (Windows の場合)
See: [here](no30599nf.html) for details

### JavaThread::cache_global_variables()  (Linux x86 の場合)
See: [here](no3059vH2.html) for details
### JavaThread::cache_global_variables()  (Linux sparc の場合)
(#Under Construction)

### JavaThread::cache_global_variables()  (Linux zero の場合)
(#Under Construction)

### JavaThread::cache_global_variables()  (Solaris sparc の場合)
(#Under Construction)

### JavaThread::cache_global_variables()  (Solaris x86 の場合)
(#Under Construction)

### JavaThread::cache_global_variables()  (Windows x86 の場合)
(#Under Construction)

### ThreadStateTransition::transition_and_fence()
See: [here](no788260p.html) for details
### JNIHandleBlock::allocate_block()
See: [here](no3718fXI.html) for details
### Thread::set_active_handles()
See: [here](no3059oAF.html) for details
### JvmtiExport::post_thread_start()
(#Under Construction)

### JavaThread::thread_main_inner()
See: [here](no2114-CM.html) for details
### JavaThread::entry_point()
See: [here](no3059ubL.html) for details
### thread_entry()
See: [here](no30597lR.html) for details
### java.lang.Thread.run()
See: [here](no3059IwX.html) for details






