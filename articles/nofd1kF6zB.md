---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： Runtime(SharedRuntime, OptoRuntime, Runtime1, SharkRuntime, Deoptimization)の呼び出し時 (JRT_*)
---
[Up](no3059XPe.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： Runtime(SharedRuntime, OptoRuntime, Runtime1, SharkRuntime, Deoptimization)の呼び出し時 (JRT_*)

--- 
## 概要(Summary)
InterpreterRuntime 以外の Runtime クラス (e.g. SharedRuntime, Runtime1, OptoRuntime, SharkRuntime) において, 
その Runtime クラス外から呼び出される可能性がある関数 (= Runtime 外からの entry point になるもの) については,
呼び出し時に明示的に SafepointSynchronize::_state のチェックが行われる.
この時点で Safepoint が開始されていた場合, そのスレッドはその場で停止する.

より具体的に言うと, 上記のような関数は定義時に以下のマクロを用いて定義されている.

  * JRT_ENTRY マクロ
  * JRT_END マクロ

これらのマクロが生成するコードから ThreadInVMfromJava のコンストラクタ/デストラクタが呼び出され,
その中で SafepointSynchronize::_state の値を確認している.

## 備考(Notes)
非常に簡単な Runtime 関数用に, Safepoint チェックを行わない JRT_LEAF というマクロも用意されている.

また, 一部の Runtime 関数のために JRT_ENTRY_NO_ASYNC というマクロも用意されている.
JRT_ENTRY マクロとの違いは, ThreadInVMfromJava ではなく ThreadInVMfromJavaNoAsyncException を用いる点
(これは, JavaThread::handle_special_runtime_exit_condition() 内での
 asynchronous exception 用の処理を省略した高速版).

また, 関数全体ではなく関数内の一部でだけ JavaThreadState を変化させたいという場合のために,
JRT_BLOCK/JRT_BLOCK_END というマクロも用意されている.

(なお JRT_BLOCK を使う場合, それを含む関数自体は JRT_BLOCK_ENTRY マクロと JRT_END マクロを使って定義する.
 これらは, 
 ただし, その必要が無ければ Deoptimization::fetch_unroll_info_helper() のように 
 JRT_BLOCK() / JRT_BLOCK_END() だけで使ってもいい.)


## 処理の流れ (概要)(Execution Flows : Summary)
### JRT_ENTRY マクロ ~ JRT_END マクロ
```
JRT_ENTRY マクロ
-> ThreadInVMfromJava::ThreadInVMfromJava()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JRT_END マクロ
(JRT_ENTRY マクロで宣言されていた ThreadInVMfromJava のデストラクタが呼ばれる)
-> ThreadInVMfromJava::~ThreadInVMfromJava()
   -> (See: [here](no8p2E6iLf.html) for details)
```


### IRT_LEAF マクロ ~ IRT_END マクロ
```
(Safepoint チェック処理はない)
```


### JRT_BLOCK マクロ ~ JRT_BLOCK_END マクロ
```
JRT_BLOCK マクロ
-> ThreadInVMfromJava::ThreadInVMfromJava()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JRT_BLOCK_END マクロ
(JRT_BLOCK マクロで宣言されていた ThreadInVMfromJava のデストラクタが呼ばれる)
-> ThreadInVMfromJava::~ThreadInVMfromJava()
   -> (See: [here](no8p2E6iLf.html) for details)
```


### JRT_ENTRY_NO_ASYNC マクロ ~ JRT_END マクロ
```
JRT_ENTRY_NO_ASYNC マクロ
-> ThreadInVMfromJavaNoAsyncException::ThreadInVMfromJavaNoAsyncException()
   -> (See: [here](no8p2E6iLf.html) for details)
```

```
JRT_END マクロ
(JRT_ENTRY_NO_ASYNC マクロで宣言されていた ThreadInVMfromJavaNoAsyncException のデストラクタが呼ばれる)
-> ThreadInVMfromJavaNoAsyncException::~ThreadInVMfromJavaNoAsyncException()
   -> (See: [here](no8p2E6iLf.html) for details)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### JRT_ENTRY マクロ
See: [here](no7882urq.html) for details
### JRT_END マクロ
See: [here](no7882IA3.html) for details
### JRT_LEAF マクロ
See: [here](no788271w.html) for details
### JRT_BLOCK マクロ
See: [here](no7882HUM.html) for details
### JRT_BLOCK_END マクロ
See: [here](no7882UeS.html) for details
### JRT_BLOCK_ENTRY マクロ
See: [here](no7882hoY.html) for details
### JRT_ENTRY_NO_ASYNC マクロ
See: [here](no78826JG.html) for details






