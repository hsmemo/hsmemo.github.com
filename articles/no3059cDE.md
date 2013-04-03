---
layout: default
title: Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： ArithmeticException の検出処理 ： Template Interpreter の場合  
---
[Up](noL4kjW5cR.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： ArithmeticException の検出処理 ： Template Interpreter の場合  

--- 
## 概要(Summary)
ArithmeticException については, 明示的にチェックしている実装とそうでない実装がある (See: [here](no3059EV1.html) for details).

  * sparc の場合: 

    idiv/irem, ldiv/lrem の全てのケースについて, 明示的にチェックしている (なんで?? #TODO).

  * x86_64 の場合: 

    idiv/irem については, シグナルハンドラで検出しているが,
    ldiv/lrem については, 明示的にチェックしている.

    (signal handler で int だったか long だったかを切り分けるのが面倒だから?? #TODO)

## 処理の流れ (概要)(Execution Flows : Summary)
(See: [here](no3059EV1.html) for details)

## 処理の流れ (詳細)(Execution Flows : Details)
(See: [here](no3059EV1.html) for details)






