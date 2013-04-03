---
layout: default
title: java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (1) ： `java.lang.Thread.<init>()` の処理
---
[Up](no_adcwNt_.html) [Top](../index.html)

#### java.lang.Thread オブジェクト (= JavaThread オブジェクト) を生成する側の処理 (1) ： `java.lang.Thread.<init>()` の処理

--- 
## 概要(Summary)
java.lang.Thread のコンストラクタは複数存在するが, 
どれも内部的には java.lang.Thread.init() を呼び出す.

この java.lang.Thread.init() 内で初期化が行われる.

## 処理の流れ (概要)(Execution Flows : Summary)
```
java.lang.Thread.<init>()
-> java.lang.Thread.init()
   -> java.lang.ThreadGroup.addUnstarted()
   -> フィールドの初期化等
```

## 処理の流れ (詳細)(Execution Flows : Details)
### `java.lang.Thread.<init>()`
See: [here](no21148yV.html) for details
### `java.lang.Thread.<init>(Runnable target)`
See: [here](no2114J9b.html) for details
### `java.lang.Thread.<init>(ThreadGroup group, Runnable target)`
See: [here](no2114WHi.html) for details
### `java.lang.Thread.<init>(String name)`
See: [here](no2114jRo.html) for details
### `java.lang.Thread.<init>(ThreadGroup group, String name)`
See: [here](no2114wbu.html) for details
### `java.lang.Thread.<init>(Runnable target, String name)`
See: [here](no21149l0.html) for details
### `java.lang.Thread.<init>(ThreadGroup group, Runnable target, String name)`
See: [here](no2114vvD.html) for details
### `java.lang.Thread.<init>(ThreadGroup group, Runnable target, String name, long stackSize)`
See: [here](no211485J.html) for details
### java.lang.Thread.init()
See: [here](no2114JEQ.html) for details
### java.lang.ThreadGroup.addUnstarted()
See: [here](no2114WOW.html) for details






