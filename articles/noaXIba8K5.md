---
layout: default
title: LoaderConstraint クラス関連のクラス (LoaderConstraintTable, LoaderConstraintEntry)
---
[Top](../index.html)

#### LoaderConstraint クラス関連のクラス (LoaderConstraintTable, LoaderConstraintEntry)

これらは, ロードしたクラスの情報を管理するためのクラス.
より具体的に言うと, Java 仮想マシン仕様の「ロード制約(loading constraints)」を管理するためのクラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).

なお, これらは SystemDictionary クラス内で(のみ)使用される補助クラス.

### 概要(Summary)
ロード制約は, 同名のクラスが異なるクラスローダによってロードされると型安全性が壊れるため, それをチェックするためのもの.
ロード制約に違反するようなクラスロードが起こると LinkageError が送出される.

JVMS の「5.3.4 Loading Constraints」を参照.

また, アルゴリズムの詳細は [以下の論文 ("Dynamic class loading in the Java virtual machine")](http://dl.acm.org/citation.cfm?id=286945) を参照, とのこと.

        Sheng Liang, Gilad Bracha, 
        "Dynamic class loading in the Java virtual machine", 
        In Proceedings of the 13th ACM SIGPLAN conference on Object-oriented programming, systems, languages, and applications (OOPSLA '98), pages 33-44, 


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.cpp))
    // Constraints on class loaders. The details of the algorithm can be
    // found in the OOPSLA'98 paper "Dynamic Class Loading in the Java
    // Virtual Machine" by Sheng Liang and Gilad Bracha.  The basic idea is
    // that the system dictionary needs to maintain a set of contraints that
    // must be satisfied by all classes in the dictionary.
    // if defining is true, then LinkageError if already in systemDictionary
    // if initiating loader, then ok if instanceKlass matches existing entry
    
    void SystemDictionary::check_constraints(int d_index, unsigned int d_hash,
```


### クラス一覧(class list)

  * [LoaderConstraintTable](#noYbA3fc2L)
  * [LoaderConstraintEntry](#noFFnUkkwA)


---
## <a name="noYbA3fc2L" id="noYbA3fc2L">LoaderConstraintTable</a>

### 概要(Summary)
SystemDictionary クラス内で使用される補助クラス (See: SystemDictionary).

SystemDictionary が管理するロード制約情報を入れておくハッシュテーブル.


```cpp
    ((cite: hotspot/src/share/vm/classfile/loaderConstraints.hpp))
    class LoaderConstraintTable : public Hashtable<klassOop> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _loader_constraints フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Constraints on class loaders
      static LoaderConstraintTable*  _loader_constraints;
```

#### 使用箇所(where its instances are used)
* resolve 処理中で呼び出される SystemDictionary::check_signature_loaders() 内
  (より正確にはそこから呼び出される SystemDictionary::add_loader_constraint() 内) で中身が追加されている.

* 蓄えられた情報は, SystemDictionary::check_constraints() や
  SystemDictionary::find_constrained_instance_or_array_klass() で参照されているように見えるが... #TODO




### 詳細(Details)
See: [here](../doxygen/classLoaderConstraintTable.html) for details

---
## <a name="noFFnUkkwA" id="noFFnUkkwA">LoaderConstraintEntry</a>

### 概要(Summary)
LoaderConstraintTable クラス内で使用される補助クラス.

LoaderConstraintTable オブジェクト内に格納されるハッシュテーブル・エントリ.


```cpp
    ((cite: hotspot/src/share/vm/classfile/loaderConstraints.hpp))
    class LoaderConstraintEntry : public HashtableEntry<klassOop> {
```




### 詳細(Details)
See: [here](../doxygen/classLoaderConstraintEntry.html) for details

---
