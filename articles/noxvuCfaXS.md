---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： FieldID の取得処理
---
[Up](no5248c5L.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： JNI によるフィールドアクセス ： FieldID の取得処理

--- 
## 該当する JNI 関数
* `GetFieldID`,
* `GetStaticFieldID`,

## 概要(Summary)
jfieldID の生成は, jfieldIDWorkaround クラスが行っている (See: jfieldIDWorkaround).

jfieldID 値のエンコード方式は以下の通り.

  * instance field の場合:
   
    最下位 1bit は 1 (これで static field と区別可能).
    上位 30bits は, フィールド位置を示すオフセット情報.
   
  * static field の場合:
   
    フィールド位置を示す JNIid オブジェクトへのポインタが jfieldID として使われる (See: JNIid).
    (このため最下位 1bit は 0 になる. これで instance field と区別可能).

...(#TODO)


```cpp
    ((cite: hotspot/src/share/vm/runtime/jfieldIDWorkaround.hpp))
      // This workaround is because JVMTI doesn't have distinct entry points
      // for methods that use static jfieldIDs and instance jfieldIDs.
      // The workaround is to steal a low-order bit:
      //   a 1 means the jfieldID is an instance jfieldID,
      //             and the rest of the word is the offset of the field.
      //   a 0 means the jfieldID is a static jfieldID,
      //             and the rest of the word is the JNIid*.
      //
      // Another low-order bit is used to mark if an instance field
      // is accompanied by an indication of which class it applies to.
      //
      // Bit-format of a jfieldID (most significant first):
      //  address:30        instance=0:1 checked=0:1
      //  offset:30         instance=1:1 checked=0:1
      //  klass:23 offset:7 instance=1:1 checked=1:1
      //
      // If the offset does not fit in 7 bits, or if the fieldID is
      // not checked, then the checked bit is zero and the rest of
      // the word (30 bits) contains only the offset.
      //
```


## 処理の流れ (概要)(Execution Flows : Summary)
### GetFieldID() の処理
```
jni_GetFieldID()
-> instanceKlass::find_field()
-> jfieldIDWorkaround::to_instance_jfieldID()
```

### GetStaticFieldID() の処理
```
jni_GetStaticFieldID()
-> instanceKlass::find_field()
-> instanceKlass::jni_id_for()
   -> instanceKlass::jni_id_for_impl()
-> jfieldIDWorkaround::to_static_jfieldID()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_GetFieldID()
See: [here](no3059OeQ.html) for details
### jfieldIDWorkaround::to_instance_jfieldID()
See: [here](no2935Ygb.html) for details
### jni_GetStaticFieldID()
See: [here](no3059boW.html) for details
### instanceKlass::jni_id_for()
See: [here](no2935y0n.html) for details
### instanceKlass::jni_id_for_impl()
See: [here](no2935MJ0.html) for details
### jfieldIDWorkaround::to_static_jfieldID()
See: [here](no2935lqh.html) for details






