---
layout: default
title: Interpreter に関する処理 ： 初期化処理 
---
[Up](nooWY3SS9t.html) [Top](../index.html)

#### Interpreter に関する処理 ： 初期化処理 

--- 
## 概要(Summary)
Interpreter の初期化処理は HotSpot の起動時に行われる.

初期化処理は大きく分けると 3段階に分けられる.
最初の2つはインタープリタ種別(Template Interpreter/C++ Interpreter)によらず共通しているが, 
最後の段階はインタープリタ種別によって異なる.

1. Bytecode table の初期化処理 (See: [here](nowPseFDvx.html) for details)

   この処理は, インタープリタ種別(Template Interpreter/C++ Interpreter)によらず共通.

2. Interpreter 自体の初期化処理 (1)  (See: [here](no3059kuA.html) for details)

   この処理は, インタープリタ種別(Template Interpreter/C++ Interpreter)によらず共通.

3. Interpreter 自体の初期化処理 (2)  (See: [here](noGVP1OhVl.html) for details)

   この処理は, インタープリタ種別(Template Interpreter/C++ Interpreter)によって異なる.




## Subcategories
* [Interpreter に関する処理 ： 初期化処理 (1) ： Bytecode table の初期化処理](nowPseFDvx.html)
* [Interpreter に関する処理 ： 初期化処理 (2) ： Interpreter 自体の初期化処理 (1) ](no3059kuA.html)
* [Interpreter に関する処理 ： 初期化処理 (3) ： Interpreter 自体の初期化処理 (2)](noGVP1OhVl.html)



