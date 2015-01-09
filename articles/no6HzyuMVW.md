---
layout: default
title: JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理
---
[Up](noQrGfj91w.html) [Top](../index.html)

#### JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理

--- 
## 概要(Summary)
JIT コンパイル処理は, 以下の契機で開始される.

  * あるメソッドの呼び出された回数が閾値を越えた際
    (See: [here](nohhZ5yDpa.html) for details)

  * あるループの実行された回数が閾値を越えた際
    (See: [here](nohhZ5yDpa.html) for details)

  * 特殊なコマンドラインオプション(※)が指定されている場合は、メソッドの呼び出し時や Constant Pool 中のメソッドの resolve 時
    (See: [here](nokhwJbLZP.html) for details)
    
    (※: -Xcomp, -XX:-UseInterpreter, -XX:+AlwaysCompileLoopMethods が指定されている場合. この場合はメソッドは常に JIT コンパイルしてから実行する)
    


## Subcategories
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： 実行回数が閾値を超えたことによる開始処理](nohhZ5yDpa.html)
* [JIT Compiler の処理 (1) ： JIT コンパイルを開始する処理 ： -Xcomp (-XX：-UseInterpreter), -XX：+AlwaysCompileLoopMethods による開始処理](nokhwJbLZP.html)



