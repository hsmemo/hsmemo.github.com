---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： クラスに関する処理(Class Operations)
---
[Up](no7882H_v.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： クラスに関する処理(Class Operations)

--- 
## 該当する JNI 関数
* `DefineClass`,
* `FindClass`,
* `GetSuperclass`,
* `IsAssignableFrom`,

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### DefineClass() の処理
```
jni_DefineClass()
-> SystemDictionary::resolve_from_stream()
   -> (See: [here](notXYWwprj.html) for details)
```

### FindClass() の処理
```
jni_FindClass()
-> find_class_from_class_loader()
   -> SystemDictionary::resolve_or_fail()
      -> (See: [here](notXYWwprj.html) for details)
-> CompilationPolicy::completed_vm_startup()  (<= 初回の呼び出し時のみ)
```

### GetSuperclass() の処理
```
jni_GetSuperclass()
-> Klass::java_super()
   (instanceKlass::java_super() と arrayKlass::java_super() でオーバーライドされている)
```

### IsAssignableFrom() の処理
```
jni_IsAssignableFrom()
-> Klass::is_subtype_of()
   -> (See: ...#TODO)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_DefineClass()
See: [here](no3059dxY.html) for details

### jni_FindClass()
(#Under Construction)
See: [here](no3059QnS.html) for details
### find_class_from_class_loader()
(#Under Construction)
See: [here](no3059EQr.html) for details
### CompilationPolicy::completed_vm_startup()
See: [here](no3059Rax.html) for details

### jni_GetSuperclass()
See: [here](no3059DdM.html) for details
### instanceKlass::java_super()
See: [here](no30593Fl.html) for details
### arrayKlass::java_super()
See: [here](no3059q7e.html) for details

### jni_IsAssignableFrom()
See: [here](no30592SG.html) for details






