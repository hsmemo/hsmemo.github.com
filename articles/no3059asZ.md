---
layout: default
title: JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合  
---
[Up](noF_QFKdsW.html) [Top](../index.html)

#### JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合  

--- 
## 概要(Summary)
JIT コンパイル対象になっていない native method の場合, 
method entry 部での処理には InterpreterGenerator::generate_native_entry() で生成されたコードが使われる
(Native メソッドの methodOop の i2i_entry には InterpreterGenerator::generate_native_entry() が生成したスタブが入っているため).



## Subcategories
* [JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合 ： Template Interpreter の場合](noPQtTMmO9.html)
* [(#TBD) JNI の処理 ： native method の処理 ： native method の呼び出し処理 ： JIT コンパイル対象になっていない native method の場合 ： C++ Interpreter の場合](noFTHTYYMO.html)



