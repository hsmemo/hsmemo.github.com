---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理
---
[Up](no6HzyuMVW.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理

--- 
## 概要(Summary)
「あるメソッドの呼び出された回数が閾値を越えた」場合, あるいは「あるループの実行された回数が閾値を越えた」場合は, 
その閾値を超えたコードに対して JIT コンパイルが開始される.

処理の流れは以下のようになる.

1. 各メソッドの呼び出し時またはコード中の各ループの繰り返し時には, 対応づけられている InvocationCounter のカウンタ値をインクリメントする処理が行われる.
   カウンタ値が閾値を超えると, JIT コンパイル処理を行うかどうかを判断するために CompilationPolicy オブジェクトが呼び出される.
   
   なお, InvocationCounter オブジェクトは methodOopDesc 及び methodDataOopDesc 内に格納されている. (See: [here](noAWg22n6P.html) for details)

2. CompilationPolicy オブジェクトによる判断は CompilationPolicy::event() で行われる.
   
   なお, 実際には CompilationPolicy オブジェクトには4つのクラスが存在し, 対応するサブクラスがオーバーライドした関数で判断が行われる.
   (See: [here](no3420bIr.html), [here](no34200pY.html), [here](no3420O-k.html) and [here](no3420B0e.html) for details)

以上の処理で CompilationPolicy オブジェクトが JIT コンパイルを行うべきと判断すれば,
CompileBroker::compile_method() が呼び出されて JIT コンパイルが開始される.
(See: [here](nobV3Ayv16.html) for details)




## Subcategories
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： メソッドの呼び出し回数のチェック処理](nolR92Ajmf.html)
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (1) 閾値のチェック処理 ： ループの実行回数のチェック処理 ](no2935sgV.html)
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理 ： (2) CompilationPolicy の処理](noi9gh3rMo.html)



