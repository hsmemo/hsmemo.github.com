---
layout: default
title: Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： NullPointerException の検出処理 ： C++ Interpreter の場合
---
[Up](no3Ny4vS28.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (1) ： 例外送出条件の検出処理 ： signal handler で検出する例外 ： NullPointerException の検出処理 ： C++ Interpreter の場合

--- 
## 概要(Summary)
C++ Interpreter の場合, Signal Handler による例外検出は行われない.


```
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    #ifdef CC_INTERP
        // C++ interpreter doesn't throw implicit exceptions
```






