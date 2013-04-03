---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI によるJavaVMインターフェースの取得処理(Java VM Interface)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI によるJavaVMインターフェースの取得処理(Java VM Interface)

--- 
## 該当する JNI 関数
* `GetJavaVM`,

## 概要(Summary)
main_vm という変数に JNIInvokeInterface_ 型の値へのポインタが格納されているため, それをリターンするだけ.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    struct JavaVM_ main_vm = {&jni_InvokeInterface};
```

## 備考(Notes)
jni_InvokeInterface は以下のように定義されている.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    const struct JNIInvokeInterface_ jni_InvokeInterface = {
        NULL,
        NULL,
        NULL,
    
        jni_DestroyJavaVM,
        jni_AttachCurrentThread,
        jni_DetachCurrentThread,
        jni_GetEnv,
        jni_AttachCurrentThreadAsDaemon
    };
```

## 処理の流れ (概要)(Execution Flows : Summary)
```
jni_GetJavaVM()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### jni_GetJavaVM()
See: [here](no3059BNW.html) for details






