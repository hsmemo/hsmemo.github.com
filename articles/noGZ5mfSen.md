---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM 関係の初期化処理
---
[Up](no2114S_x.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM 関係の初期化処理

--- 
## 概要(Summary)
JMM 関係の初期化処理には以下の 3種類がある.

* メモリ関係の初期化処理
  
  java.lang.management.MemoryManagerMXBean 及び
  java.lang.management.MemoryPoolMXBean を実現するための処理.
  
* JMM 用のネイティブライブラリの初期化処理
  
  jmm interface (See: [here](noRM-0G7af.html) for details) 関係の初期化を行う処理.

* JMX Management Server の起動処理
  
  JMX Management Server (Platform MXBean server) 関係の初期化を行う処理.




## Subcategories
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM のメモリ関係の初期化処理  ](no211477i.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM 用のネイティブライブラリの初期化処理 (JMM Interface の取得処理)  ](no7882-WA.html)
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMX Management Server の起動処理](novYWKneN9.html)



