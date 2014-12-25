---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイム呼び出し時の Safepoint 停止処理 (IRT_*, JRT_*)  
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイム呼び出し時の Safepoint 停止処理 (IRT_*, JRT_*)  

--- 
## 概要(Summary)
Runtime の関数の中で
Runtime 外から呼び出される可能性があるもの (= Runtime 外からの entry point になるもの) については,
呼び出し時に明示的に SafepointSynchronize::_state のチェックが行われる.
この時点で Safepoint が開始されていた場合, そのスレッドはその場で停止する.

より具体的に言うと, 上記のような Runtime の関数は定義時に以下のマクロを用いて定義されている.

  * IRT_ENTRY マクロ, IRT_END マクロ, etc
  * JRT_ENTRY マクロ, JRT_END マクロ, etc

これらのマクロが生成するコードから ThreadInVMfromJava のコンストラクタ/デストラクタが呼び出され,
その中で SafepointSynchronize::_state の値を確認している.

## 備考(Notes)
IRT_ENTRY は InterpreterRuntime 向け, JRT_ENTRY はその他の Runtime 向け
(See: InterpreterRuntime, SharedRuntime, Runtime1, OptoRuntime, SharkRuntime).


```cpp
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // Definitions for IRT (Interpreter Runtime)
    // (thread is an argument passed in to all these routines)
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.hpp))
    // Definitions for JRT (Java (Compiler/Shared) Runtime)
```




## Subcategories
* [Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： InterpreterRuntime の呼び出し時 (IRT_*)  ](no7882haw.html)
* [Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ランタイムの呼び出し時 ： Runtime(SharedRuntime, OptoRuntime, Runtime1, SharkRuntime, Deoptimization)の呼び出し時 (JRT_*)](nofd1kF6zB.html)



