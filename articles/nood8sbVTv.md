---
layout: default
title: ResolutionError クラス関連のクラス (ResolutionErrorTable, ResolutionErrorEntry)
---
[Top](../index.html)

#### ResolutionError クラス関連のクラス (ResolutionErrorTable, ResolutionErrorEntry)

これらは, ロードしたクラスの情報を管理するためのクラス.
より具体的に言うと, リンク中の解決処理(resolution)で生じたエラーを記録しておくためのクラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).

なお, これらは SystemDictionary クラス内で(のみ)使用される補助クラス.

### 概要(Summary)
リンク中の解決処理(resolution)に付いては JVMS 5.4.3 参照.

* ResolutionErrorEntry は, 1つ1つのエラーを表現するもの.
* ResolutionErrorTable は, ResolutionErrorEntry を入れておくハッシュテーブル
  (constantPoolHandle と constant pool index をキーとして, 対応する ResolutionErrorEntry を返す).


```
    ((cite: hotspot/src/share/vm/classfile/resolutionErrors.hpp))
    // ResolutionError objects are used to record errors encountered during
    // constant pool resolution (JVMS 5.4.3).
```


### クラス一覧(class list)

  * [ResolutionErrorTable](#no95bjMpwH)
  * [ResolutionErrorEntry](#nomtYc47oA)


---
## <a name="no95bjMpwH" id="no95bjMpwH">ResolutionErrorTable</a>

### 概要(Summary)
SystemDictionary クラス内で使用される補助クラス (See: SystemDictionary).

SystemDictionary が管理する「リンク中の解決処理(resolution)で生じたエラー情報」を入れておくハッシュテーブル.


```
    ((cite: hotspot/src/share/vm/classfile/resolutionErrors.hpp))
    class ResolutionErrorTable : public Hashtable<constantPoolOop> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _resolution_errors フィールド (static フィールド) に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Resolution errors
      static ResolutionErrorTable*   _resolution_errors;
```




### 詳細(Details)
See: [here](../doxygen/classResolutionErrorTable.html) for details

---
## <a name="nomtYc47oA" id="nomtYc47oA">ResolutionErrorEntry</a>

### 概要(Summary)
ResolutionErrorTable クラス内で使用される補助クラス.

ResolutionErrorTable オブジェクト内に格納されるハッシュテーブル・エントリ.


```
    ((cite: hotspot/src/share/vm/classfile/resolutionErrors.hpp))
    class ResolutionErrorEntry : public HashtableEntry<constantPoolOop> {
```





### 詳細(Details)
See: [here](../doxygen/classResolutionErrorEntry.html) for details

---
