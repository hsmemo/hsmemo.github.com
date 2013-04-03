---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： バージョン情報(Version Information)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： バージョン情報(Version Information)

--- 
## 該当する JNI 関数
* `GetVersion`

## 概要(Summary)
バージョン情報は CurrentVersion という大域変数に格納されているため, それをリターンするだけ.

## 処理の流れ (概要)(Execution Flows : Summary)
### GetVersion() の処理
```
jni_GetVersion()
-> (CurrentVersion 変数の値をリターンするだけ)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_GetVersion()
See: [here](no3059ok0.html) for details






