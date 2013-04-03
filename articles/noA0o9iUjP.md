---
layout: default
title: StackMapTable クラス関連のクラス (StackMapTable, StackMapStream, StackMapReader)
---
[Top](../index.html)

#### StackMapTable クラス関連のクラス (StackMapTable, StackMapStream, StackMapReader)

これらは, クラスファイルの verify 処理用のクラス.
より具体的に言うと, クラスファイル中の StackMapTable 情報 (StackMap attribute) を扱うためのクラス (See: [here](no7882amm.html) for details).

### 概要(Summary)
クラスファイル内には StackMapTable attrible が入っており, その中には複数の stack_map_frame が納められている
(JVMS 4.7.4 参照).
これらのクラスは (StackMapFrame クラスと併せて) 以下の役割を担当する.

 * StackMapTable は, クラスファイル中の StackMapTable attrible を表すクラス.

 * StackMapFrame は, StackMapTable attrible 中の stack_map_frame を表すクラス.

 * StackMapReader は, StackMapTable attribute の読み出し処理を行うクラス

 * StackMapStream は, StackMapReader 内部で使用される補助クラス. 実際に読み出し処理を行う.



### クラス一覧(class list)

  * [StackMapTable](#noKDBU_OtI)
  * [StackMapReader](#nolni7UE2s)
  * [StackMapStream](#no9vzUc8yp)


---
## <a name="noKDBU_OtI" id="noKDBU_OtI">StackMapTable</a>

### 概要(Summary)
ClassVerifier クラス内で使用される補助クラス.
クラスファイルの verify 処理で使用される一時オブジェクト(StackObjクラス).

クラスファイル中の StackMapTable attrible を表す.


```
    ((cite: hotspot/src/share/vm/classfile/stackMapTable.hpp))
    // StackMapTable class is the StackMap table used by type checker
    class StackMapTable : public StackObj {
```

### 使われ方(Usage)
ClassVerifier::verify_method() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classStackMapTable.html) for details

---
## <a name="nolni7UE2s" id="nolni7UE2s">StackMapReader</a>

### 概要(Summary)
ClassVerifier クラス内で使用される補助クラス.
クラスファイルの verify 処理で使用される一時オブジェクト(StackObjクラス).

クラスファイル中の StackMapTable attrible 情報の読み出しを行う.


```
    ((cite: hotspot/src/share/vm/classfile/stackMapTable.hpp))
    class StackMapReader : StackObj {
```

### 使われ方(Usage)
ClassVerifier::verify_method() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classStackMapReader.html) for details

---
## <a name="no9vzUc8yp" id="no9vzUc8yp">StackMapStream</a>

### 概要(Summary)
StackMapReader クラス用の補助クラス.
クラスファイルの verify 処理で使用される一時オブジェクト(StackObjクラス).

実際の読み出し処理を行う.


```
    ((cite: hotspot/src/share/vm/classfile/stackMapTable.hpp))
    class StackMapStream : StackObj {
```

### 使われ方(Usage)
ClassVerifier::verify_method() 内で(のみ)使用されている.

(より正確に言うと, ClassVerifier::verify_method() から呼び出される StackMapReader のメソッド中でのみ使用される)




### 詳細(Details)
See: [here](../doxygen/classStackMapStream.html) for details

---
