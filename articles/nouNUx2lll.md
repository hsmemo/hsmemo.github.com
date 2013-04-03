---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： 低速アクセス処理
---
[Up](no5248c5L.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： 低速アクセス処理

--- 
## 該当する JNI 関数
* `Get<type>Field`,
* `Set<type>Field`,
* `GetStatic<type>Field`,
* `SetStatic<type>Field`,


## 概要(Summary)
(#Under Construction)

処理用の関数のうち型が Object でないものについては, 以下のマクロを用いて定義されている.
(Object のものについてだけは, マクロを使わずに単体で定義されている)

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table9282ATX -->
| Function | Macro |
|---|---|
| `Get<type>Field()` | DEFINE_GETFIELD() マクロ |
| `Set<type>Field()` | DEFINE_SETFIELD() マクロ |
| `GetStatic<type>Field()` | DEFINE_GETSTATICFIELD() マクロ |
| `SetStatic<type>Field()` | DEFINE_SETSTATICFIELD() マクロ |
<!-- END RECEIVE ORGTBL table9282ATX -->

<!-- 
#+ORGTBL: SEND table9282ATX orgtbl-to-gfm :no-escape t
| Function                 | Macro                          |
|--------------------------+--------------------------------|
| `Get<type>Field()`       | DEFINE_GETFIELD() マクロ       |
| `Set<type>Field()`       | DEFINE_SETFIELD() マクロ       |
| `GetStatic<type>Field()` | DEFINE_GETSTATICFIELD() マクロ |
| `SetStatic<type>Field()` | DEFINE_SETSTATICFIELD() マクロ |
-->


## 処理の流れ (概要)(Execution Flows : Summary)
### Get<type>Field() 用の処理
```
DEFINE_GETFIELD() マクロ  or  jni_GetObjectField()
-> jfieldIDWorkaround::from_instance_jfieldID()
-> oopDesc::<type>_field()
```

### Set<type>Field() 用の処理
```
DEFINE_SETFIELD() マクロの処理  or  jni_SetObjectField()
-> jfieldIDWorkaround::from_instance_jfieldID()
-> oopDesc::<type>_field_put()
```

### GetStatic<type>Field() 用の処理
```
DEFINE_GETSTATICFIELD() マクロの処理  or  jni_GetStaticObjectField()
-> jfieldIDWorkaround::from_static_jfieldID()
-> oopDesc::<type>_field()
```

### SetStatic<type>Field() 用の処理
```
DEFINE_SETSTATICFIELD() マクロの処理  or  jni_SetStaticObjectField()
-> jfieldIDWorkaround::from_static_jfieldID()
-> oopDesc::<type>_field_put()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### DEFINE_GETFIELD() マクロ
(#Under Construction)
See: [here](no3059CHp.html) for details
### jni_GetObjectField()
See: [here](no305918i.html) for details
### DEFINE_SETFIELD() マクロ
(#Under Construction)
See: [here](no3059cb1.html) for details
### jni_SetObjectField()
See: [here](no3059PRv.html) for details
### DEFINE_GETSTATICFIELD() マクロ
(#Under Construction)
See: [here](no3059bvK.html) for details
### jni_GetStaticObjectField()
See: [here](no3059OlE.html) for details
### DEFINE_SETSTATICFIELD() マクロ
(#Under Construction)
See: [here](no30591DX.html) for details
### jni_SetStaticObjectField()
See: [here](no3059o5Q.html) for details






