---
layout: default
title: Dictionary クラス関連のクラス (Dictionary, ProtectionDomainEntry, DictionaryEntry, SymbolPropertyEntry, SymbolPropertyTable)
---
[Top](../index.html)

#### Dictionary クラス関連のクラス (Dictionary, ProtectionDomainEntry, DictionaryEntry, SymbolPropertyEntry, SymbolPropertyTable)

これらは, ロードしたクラスの情報を管理するためのクラス.
より具体的に言うと, SystemDictionary クラス用の補助クラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).


### クラス一覧(class list)

  * [Dictionary](#noPwe0dS2O)
  * [ProtectionDomainEntry](#noHWkQQERf)
  * [DictionaryEntry](#no4rTABitN)
  * [SymbolPropertyTable](#nouA6XelX8)
  * [SymbolPropertyEntry](#noqXZas4GS)


---
## <a name="noPwe0dS2O" id="noPwe0dS2O">Dictionary</a>

### 概要(Summary)
SystemDictionary クラス内で使用される補助クラス (See: SystemDictionary).

SystemDictionary が管理するクラス情報を入れておくハッシュテーブル.


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
    //~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    // The data structure for the system dictionary (and the shared system
    // dictionary).
    
    class Dictionary : public TwoOopHashtable<klassOop> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* SystemDictionary クラスの _disctionary フィールド (static フィールド)
  
  (ロードしたクラス用)

* SystemDictionary クラスの _shared_dictionary フィールド (static フィールド)
  
  (shared archive から取得したクラス用 (なお, shared archive とは Class Data Sharing (CDS) の領域のこと))


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Hashtable holding loaded classes.
      static Dictionary*            _dictionary;
    ...
      // Hashtable holding classes from the shared archive.
      static Dictionary*             _shared_dictionary;
```

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* SystemDictionary::initialize()
  
  (SystemDictionary::_disctionary フィールドの初期化用)

* SystemDictionary::set_shared_dictionary()

  (SystemDictionary::_shared_dictionary フィールドの初期化用)




### 詳細(Details)
See: [here](../doxygen/classDictionary.html) for details

---
## <a name="noHWkQQERf" id="noHWkQQERf">ProtectionDomainEntry</a>

### 概要(Summary)
Dictionary クラス内で使用される補助クラス.

Java の "ProtectionDomain" 情報を管理するためのクラス.
より具体的に言うと, java.security.ProtectionDomain オブジェクトを格納するための線形リスト.

(SystemDictionary の内部では, ロード済みのクラスは { klassOop, loader, protection_domain } という3つ組で管理している
 (See: DictionaryEntry).
そして, 既にロード済みのクラスと protection_domain が合わなければ例外を出す)


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
    // The following classes can be in dictionary.cpp, but we need these
    // to be in header file so that SA's vmStructs can access.
    
    class ProtectionDomainEntry :public CHeapObj {
```

### 内部構造(Internal structure)
以下のようなフィールドからなる
(要は _next でつながっていく線形リストになっている).

_protection_domain フィールドに, 実際の java.security.ProtectionDomain オブジェクトが入る.


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
      ProtectionDomainEntry* _next;
      oop                    _protection_domain;
```

(このリストに一度 validate が通った java.security.ProtectionDomain オブジェクトをキャッシュしておくことで,
次回以降の validate を高速化するために使われている)

(なお, このリストにキャッシュされて無かった場合は,
クラスローダーの checkPackageAccess() メソッドでチェックが行われる.
そして, チェックに通れば DictionaryEntry::add_protection_domain() でこのリストに追加される)

### 備考(Notes)
develop オプションである ProtectionDomainVerification を false にすると,
protection domain チェックを無効にすることもできる.




### 詳細(Details)
See: [here](../doxygen/classProtectionDomainEntry.html) for details

---
## <a name="no4rTABitN" id="no4rTABitN">DictionaryEntry</a>

### 概要(Summary)
Dictionary クラス内で使用される補助クラス.

Dictionary オブジェクト内に格納されるハッシュテーブル・エントリ.


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
    // An entry in the system dictionary, this describes a class as
    // { klassOop, loader, protection_domain }.
    
    class DictionaryEntry : public HashtableEntry<klassOop> {
```

### 内部構造(Internal structure)
スーパークラスである HashtableEntry クラスのフィールドに加えて, 以下のフィールドを持つ.

* oop                    _loader

  そのクラスをロードしたクラスローダを記憶しておくフィールド

* ProtectionDomainEntry* _pd_set
  
  一度 validate が通った ProtectionDomainEntry をキャッシュしておくフィールド

(なお, 実際のクラスオブジェクトはスーパークラスである HashtableEntry のフィールド中に格納されている)


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
      // Contains the set of approved protection domains that can access
      // this system dictionary entry.
      ProtectionDomainEntry* _pd_set;
      oop                    _loader;
```




### 詳細(Details)
See: [here](../doxygen/classDictionaryEntry.html) for details

---
## <a name="nouA6XelX8" id="nouA6XelX8">SymbolPropertyTable</a>

### 概要(Summary)
SystemDictionary クラス内で使用される補助クラス (See: SystemDictionary).

java.lang.invoke.MethodHandle.invoke() の処理用のハッシュテーブル.
メソッド名と型(Signature String)を表す Symbol オブジェクトをキーとして対応する methodOopDesc を引くことが出来る.


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
    // A system-internal mapping of symbols to pointers, both managed
    // and unmanaged.  Used to record the auto-generation of each method
    // MethodHandle.invoke(S)T, for all signatures (S)T.
    class SymbolPropertyTable : public Hashtable<Symbol*> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _invoke_method_table フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
SystemDictionary::initialize() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
SystemDictionary::find_method_handle_invoke() 内で使用されている.




### 詳細(Details)
See: [here](../doxygen/classSymbolPropertyTable.html) for details

---
## <a name="noqXZas4GS" id="noqXZas4GS">SymbolPropertyEntry</a>

### 概要(Summary)
SymbolPropertyTable クラス内で使用される補助クラス.

SymbolPropertyTable オブジェクト内に格納されるハッシュテーブル・エントリ.


```cpp
    ((cite: hotspot/src/share/vm/classfile/dictionary.hpp))
    // Entry in a SymbolPropertyTable, mapping a single Symbol*
    // to a managed and an unmanaged pointer.
    class SymbolPropertyEntry : public HashtableEntry<Symbol*> {
```




### 詳細(Details)
See: [here](../doxygen/classSymbolPropertyEntry.html) for details

---
