---
layout: default
title: JNI の処理 ： Invocation API の処理 ： JNI_GetCreatedJavaVMs() の処理
---
[Up](nopXLc6YjR.html) [Top](../index.html)

#### JNI の処理 ： Invocation API の処理 ： JNI_GetCreatedJavaVMs() の処理

--- 
## 概要(Summary)
HotSpot が構築済みであれば, 個数としては 1 がリターンされ, ポインタとしては作成した VM がリターンされる.
まだ構築されていなければ, 個数として 0 がリターンされる.

## 備考(Notes)
なお HotSpot の場合は個数が 2 以上になることはない模様.
これは, JNI_CreateJavaVM() の中で vm_created を確認し,
1つのプロセス内には最大1つしか VM は作らせないように制御しているため (See: [here](no2114J7x.html) for details).

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
JNI_GetCreatedJavaVMs()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### JNI_GetCreatedJavaVMs()
See: [here](no17119gEz.html) for details






