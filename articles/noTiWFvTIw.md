---
layout: default
title: JNI の処理 (Java Native Interface)
---
[Up](no1S0Auo49.html) [Top](../index.html)

#### JNI の処理 (Java Native Interface)

--- 
## 概要(Summary)
JNI 関係の処理には以下の 3種類がある.

  * native method の処理

    native method のロード／リンク処理, 及び native method の呼び出し処理

  * JNI Functions の処理

    native method から使用される "JNI Functions" の処理

  * Invocation API の処理

    native method から使用される "Invocation API" の処理

## 備考(Notes)
(参考: <http://openjdk.java.net/groups/hotspot/docs/RuntimeOverview.html> : HotSpot Runtime Overview)

* HotSpot 内部では JNI で使用される機構と, 普段(インタープリタなどから)使用される機構はまったく同じもの.

* JNI のデバッグ用に -Xcheck:jni が提供されている
  (JNI 呼び出しの際の検査がより厳しくなる).

* JNI でネイティブコードを実行しているスレッドは safepoint 中でも停止する必要はない.
  ただし, JVM に戻ろうとしたり JNI コールを呼び出したりすると停止させられる.




## Subcategories
* [JNI の処理 ： native method の処理](noNisy_uNv.html)
* [JNI の処理 ： JNI Functions の処理  ](no7882H_v.html)
* [JNI の処理 ： Invocation API の処理](nopXLc6YjR.html)



