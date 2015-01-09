---
layout: default
title: JIT Compiler の処理 (0) ： JIT 関係の初期化処理
---
[Up](noQrGfj91w.html) [Top](../index.html)

#### JIT Compiler の処理 (0) ： JIT 関係の初期化処理

--- 
## 概要(Summary)
JIT コンパイル関係のデータ構造は HotSpot の起動時に初期化される.
この初期化作業では以下のような処理が行われる.

* コンパイラオブジェクトが生成される (AbstractCompilerのサブクラスのインスタンス)
* JIT コンパイルを行うスレッドが生成される (CompilerThread)
* JIT コンパイルを行うべきかどうか判断するオブジェクトが生成される (CompilationPolicy オブジェクト)
* 

## 備考(Notes)
* CompilationPolicy オブジェクトには複数の種別が存在する.
  生成する CompilationPolicy オブジェクトの種別は, コマンドラインオプションの `-XX:CompilationPolicyChoice=<n>` で指定できる.

* CompilerThread スレッドはそれぞれの JIT コンパイルレベル用(C1 用, C2 用) に複数個生成される.
  それぞれのレベル用に何個作るかは, コマンドラインオプションで指定できる.
  数は CompilationPolicy::initialize() (を各サブクラスがオーバーライドしたもの) で算出されている.
  (See: CICompilerCount, CICompilerCountPerCPU)

## 処理の流れ (概要)(Execution Flows : Summary)
### CompilerThread の初期化処理
* CompilerThread スレッドの生成処理

<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; CompileBroker::compilation_init()
      -&gt; (1) コンパイラオブジェクト (AbstractCompilerのサブクラスのインスタンス) を生成する
             生成するコンパイラオブジェクトのクラスは HotSpot のビルド時のマクロ定義によって決まる
             * (#ifndef SHARK かつ) #ifdef COMPILER1 の場合:
               -&gt; Compiler::Compiler()
             * (#ifndef SHARK かつ) #ifdef COMPILER1 の場合:
               -&gt; C2Compiler::C2Compiler()
             * #ifdef SHARK の場合:
               -&gt; SharkCompiler::SharkCompiler()

         (1) CompilerThread を生成し, 実行を開始させる
             -&gt; CompileBroker::init_compiler_threads()
                -&gt; CompileBroker::make_compiler_thread()
                   -&gt; CompilerThread::CompilerThread()
                     -&gt; JavaThread::JavaThread() (← なお, エントリポイントとしては compiler_thread_entry() 関数が指定されている)
                        -&gt; (See: <a href="no2935KMw.html">here</a> for details)
                   -&gt; Thread::start()
                      -&gt; (See: <a href="no2935KMw.html">here</a> for details)
</pre></div>

* 生成された CompilerThread スレッドの処理

<div class="flow-abst"><pre>
compiler_thread_entry()
-&gt; CompileBroker::compiler_thread_loop()
   -&gt; (See: <a href="nobV3Ayv16.html">here</a> for details)
</pre></div>


### CompilationPolicy オブジェクトの初期化処理
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; compilationPolicy_init()
         -&gt; (1) CompilationPolicyChoice の値に応じた CompilationPolicy オブジェクトを生成する
                * 0 の場合:
                  -&gt; SimpleCompPolicy::SimpleCompPolicy()
                * 1 の場合: (#ifdef COMPILER2 でないとエラー)
                  -&gt; StackWalkCompPolicy::StackWalkCompPolicy()
                * 2 の場合: (#ifdef TIERED でないとエラー)
                  -&gt; SimpleThresholdPolicy::SimpleThresholdPolicy()   
                * 3 の場合: (#ifdef TIERED でないとエラー)
                  -&gt; AdvancedThresholdPolicy::AdvancedThresholdPolicy()

            (1) 生成した CompilationPolicy オブジェクトを初期化する
                -&gt; CompilationPolicy::initialize() (を各サブクラスがオーバーライドしたもの)
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### CompileBroker::compilation_init()
See: [here](no285644c2.html) for details
### CompileBroker::init_compiler_threads()
See: [here](no28564JA1.html) for details
### CompileBroker::make_compiler_thread()
See: [here](no28564j3F.html) for details
### CompilerThread::CompilerThread()
See: [here](no28564-aB.html) for details
### compiler_thread_entry()
See: [here](no28564yKO.html) for details
### compilationPolicy_init()
See: [here](no28564yRC.html) for details
### NonTieredCompPolicy::initialize()
See: [here](no28564AdP.html) for details
### AdvancedThresholdPolicy::initialize()
See: [here](no28564o10.html) for details
### SimpleThresholdPolicy::initialize()
See: [here](no28564BQu.html) for details





