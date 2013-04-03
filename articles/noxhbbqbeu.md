---
layout: default
title: JNI の処理 ： Invocation API の処理 ： JNI_GetDefaultJavaVMInitArgs() の処理
---
[Up](nopXLc6YjR.html) [Top](../index.html)

#### JNI の処理 ： Invocation API の処理 ： JNI_GetDefaultJavaVMInitArgs() の処理

--- 
## 概要(Summary)
処理としては, 指定されたバージョンが JNI 1.2, JNI 1.4, JNI 1.6 のどれかであれば JNI_OK を返し, それ以外なら JNI_ERR を返すだけ.


なお, 指定されたバージョンが 1.1(HotSpot はもうサポートしない) の場合には,
引数の JDK1_1InitArgs 内に以下の情報を書き込んで launcher に伝えている.

  * バージョンについては, 最も近いサポート対象バージョンである 1.2 を書き込む.
  * javaStackSize についても, デフォルトのスタックサイズを書き込む.

## 処理の流れ (概要)(Execution Flows : Summary)
```
JNI_GetDefaultJavaVMInitArgs()
-> Threads::is_supported_jni_version()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### JNI_GetDefaultJavaVMInitArgs()
See: [here](no171195lg.html) for details
### Threads::is_supported_jni_version()
See: [here](no17119Gwm.html) for details






