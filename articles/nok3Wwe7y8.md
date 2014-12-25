---
layout: default
title: jniCheck クラス 
---
[Top](../index.html)

#### jniCheck クラス 



---
## <a name="noVqy0RFqB" id="noVqy0RFqB">jniCheck</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する product オプションが指定されている場合にのみ使用される) (See: CheckJNICalls).

「ランタイムチェック付きの JNI 関数」のための補助関数を納めた名前空間(AllStatic クラス).

(なお, ランタイムチェック付きの JNI 関数は -Xcheck:jni オプション (CheckJNICalls オプション) が指定された場合にのみ使用される.
(See: checked_jni_NativeInterface) (See: [here](no5248dl2.html) and [here](no3059pIA.html) for details))


```cpp
    ((cite: hotspot/src/share/vm/prims/jniCheck.hpp))
    //
    // Checked JNI routines that are useful for outside of checked JNI
    //
    
    class jniCheck : public AllStatic {
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jniCheck.hpp))
      static oop validate_handle(JavaThread* thr, jobject obj);
      static oop validate_object(JavaThread* thr, jobject obj);
      static klassOop validate_class(JavaThread* thr, jclass clazz, bool allow_primitive = false);
      static void validate_class_descriptor(JavaThread* thr, const char* name);
      static void validate_throwable_klass(JavaThread* thr, klassOop klass);
      static void validate_call_object(JavaThread* thr, jobject obj, jmethodID method_id);
      static void validate_call_class(JavaThread* thr, jclass clazz, jmethodID method_id);
      static methodOop validate_jmethod_id(JavaThread* thr, jmethodID method_id);
```




### 詳細(Details)
See: [here](../doxygen/classjniCheck.html) for details

---
