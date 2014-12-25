---
layout: default
title: jfieldIDWorkaround クラス 
---
[Top](../index.html)

#### jfieldIDWorkaround クラス 



---
## <a name="noUKJLKAZS" id="noUKJLKAZS">jfieldIDWorkaround</a>

### 概要(Summary)
JNI 及び JVMTI の機能を実装するためのクラス 
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

jfieldID を扱うためのクラス (つまり, JNI/JVMTI によるフィールドアクセス処理で使用される補助クラス).
jfieldID の生成や jfieldID から対応するフィールドへの変換を行う
(See: [here](no5248c5L.html), [here](noxvuCfaXS.html) and [here](nouNUx2lll.html) for details).

(なお, static フィールドの場合は JNIid オブジェクトも併用される (See: JNIid))


```cpp
    ((cite: hotspot/src/share/vm/runtime/jfieldIDWorkaround.hpp))
    class jfieldIDWorkaround: AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 以下のメソッドで jfieldID 値を生成できる.
   
   * jfieldIDWorkaround::to_instance_jfieldID()
   * jfieldIDWorkaround::to_static_jfieldID()
   * jfieldIDWorkaround::to_jfieldID()

   (jfieldIDWorkaround::to_jfieldID() は, 引数の boolean 値に応じて 
   instance フィールド用か static フィールド用かが変わる)

2. 以下のメソッドで, jfieldID 値から対応する intptr_t または JNIid を取得できる.
   
   * jfieldIDWorkaround::from_instance_jfieldID()
   * jfieldIDWorkaround::from_static_jfieldID()





### 詳細(Details)
See: [here](../doxygen/classjfieldIDWorkaround.html) for details

---
