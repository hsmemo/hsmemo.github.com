---
layout: default
title: JIT Compiler の処理 (2) ： JIT コンパイラを呼び出す処理
---
[Up](noQrGfj91w.html) [Top](../index.html)

#### JIT Compiler の処理 (2) ： JIT コンパイラを呼び出す処理

--- 
## 概要(Summary)
JIT コンパイルの開始処理からは, 最終的に CompileBroker::compile_method() が呼び出される (この関数が JIT コンパイル処理のエントリポイント) (See: [here](no6HzyuMVW.html) for details).

CompileBroker::compile_method() では, ネイティブメソッドかどうかで処理が 2通りに分かれる.

  * ネイティブメソッドの場合には, AdapterHandlerLibrary::create_native_wrapper() を呼び出すことで JIT コンパイルを実行する(See: [here](no293548G.html) for details).

  * ネイティブメソッドではない場合, JIT コンパイル作業は専用のスレッド(CompilerThread)で行われる.
    
    この場合, CompileBroker::compile_method() は, CompileQueue 経由で CompilerThread にコンパイル要求(CompileTask オブジェクト)を通知する.
    
    CompilerThread は,
    CompileBroker::compiler_thread_loop() 内の無限ループ (while true ループ) で CompileTask が来るのを待っており,
    要求が来ると JIT コンパイル処理を開始する.

    実際の JIT コンパイル処理は, CompilerThread が AbstractCompiler::compile_method() (を各サブクラスがオーバーライドしたもの) を呼び出すことで実行される (See: [here](no-wa6OPxh.html) for details).

## 備考(Notes)
なお, CompilerThread は, それぞれの JIT コンパイルレベル用(C1 用, C2 用) に複数個生成されている (See: [here](nopbZ_pxeK.html) for details).
これらはそれぞれ, 対応するレベル用の CompileQueue を監視している.
コンパイル要求を伝える際には, 望みのレベルのキューに要求を入れることで, 適切な CompilerThread に通知している.


## 処理の流れ (概要)(Execution Flows : Summary)
### CompileBroker::compile_method() での処理
<div class="flow-abst"><pre>
CompileBroker::compile_method()
-&gt; * 対象がネイティブメソッドの場合:
     -&gt; AdapterHandlerLibrary::create_native_wrapper()
        -&gt; (See: <a href="no293548G.html">here</a> for details)
   * 対象がネイティブメソッドではない場合:
     -&gt; CompileBroker::compile_method_base()
        -&gt; (1) コンパイル要求をキューに詰めて CompilerThread に通知する.
               -&gt; CompileBroker::create_compile_task()
                  -&gt; CompileQueue::add()
                     -&gt; methodOopDesc::set_queued_for_compilation()
                     -&gt; Monitor::notify_all()                        &lt;= CompilerThread に通知        
           (1) (&quot;-Xbatch&quot; コマンドラインオプション等が付いている場合は) JIT コンパイル処理が終わるまで待つ
               -&gt; CompileBroker::wait_for_completion()  
</pre></div>

### 通知された CompilerThread スレッド側の処理
<div class="flow-abst"><pre>
(See: <a href="nopbZ_pxeK.html">here</a> for details)
-&gt; CompileBroker::compiler_thread_loop()
   -&gt; (1) キューから新しいコンパイル要求を取得する (キューになければ届くまで待つ)
          -&gt; CompileQueue::get()
             -&gt; Monitor::wait()                          &lt;= CompileTask が届くまで待機
             -&gt; CompilationPolicy::select_task() (を各サブクラスがオーバーライドしたもの)
             -&gt; CompileQueue::remove()
      (1) JIT コンパイルを実行する
          -&gt; CompileBroker::invoke_compiler_on_method()
             -&gt; (1) 適切なコンパイラオブジェクト (AbstractCompilerのサブクラスのインスタンス) を取得する
                    -&gt; CompileBroker::compiler()
                (1) 取得したコンパイラオブジェクトを用いて JIT コンパイル処理を行う.
                    -&gt; AbstractCompiler::compile_method() (を各サブクラスがオーバーライドしたもの)
                       -&gt; (See: <a href="no-wa6OPxh.html">here</a> for details)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### CompileBroker::compile_method()
See: [here](no6348Kaw.html) for details
### CompileBroker::compile_method_base()
See: [here](no304284z2.html) for details
### CompileBroker::compilation_is_in_queue()
See: [here](no30428ixu.html) for details
### methodOopDesc::queued_for_compilation()
See: [here](no30428Xp2.html) for details
### CompileBroker::create_compile_task()
See: [here](no18682xMK.html) for details
### CompileQueue::add()
See: [here](no18682zHk.html) for details
### methodOopDesc::set_queued_for_compilation()
See: [here](no18682nTB.html) for details

### CompileBroker::compiler_thread_loop()
See: [here](no10981QLG.html) for details
### CompileQueue::get()
See: [here](no10981TJR.html) for details
### NonTieredCompPolicy::select_task()
See: [here](no10981VLf.html) for details
### SimpleThresholdPolicy::select_task()
See: [here](no10981VST.html) for details
### AdvancedThresholdPolicy::select_task()
(#Under Construction)

### CompileBroker::invoke_compiler_on_method()
See: [here](no10981z9t.html) for details
### CompileBroker::compiler()
See: [here](no109814lV.html) for details
### is_c2_compile()
See: [here](no10981fLc.html) for details
### is_c1_compile()
See: [here](no109815mc.html) for details






