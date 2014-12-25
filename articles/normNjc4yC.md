---
layout: default
title: Placeholder クラス関連のクラス (PlaceholderTable, SeenThread, PlaceholderEntry)
---
[Top](../index.html)

#### Placeholder クラス関連のクラス (PlaceholderTable, SeenThread, PlaceholderEntry)

これらは, クラスローディング処理用のクラス.
より具体的に言うと, 現在ロード中のクラスを一時的に覚えておくためのクラス (ClassCircularityError の検査等に使用される) 
(See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).

なお, これらは SystemDictionary クラス内で(のみ)使用される補助クラス.

### 概要(Summary)
これらのオブジェクトは, ロードの開始時に作られ, ロードが完了すると破棄される (詳細は SystemDictionary クラスの説明を参照).

現在進行中のロード作業1つ1つは (class_name, loader) というタプルで表現される.
これらのクラスは以下の様に対応する.

 * PlaceholderEntry は, このタプルを表現するクラス.

 * PlaceholderTable は, PlaceholderEntry を入れておくハッシュテーブル (現在進行中のロード作業全体を示す). 
   class_name と loader をキーとして, 対応する PlaceholderEntry があるかどうかを返す.

 * SeenThread は, ある特定のクラスに対して, 現在ロードを行っているスレッド全員を表現するクラス.
   ClassCircularityError の検査等に使われる
   (例えば, superclass のロード前にこれに登録しておくことで, circularity なのか他のスレッドがロード作業中なのかが区別できる).


```cpp
    ((cite: hotspot/src/share/vm/classfile/placeholders.hpp))
    // Placeholder objects. These represent classes currently
    // being loaded, as well as arrays of primitives.
    //
```



### クラス一覧(class list)

  * [PlaceholderTable](#noxTvQ00ZA)
  * [PlaceholderEntry](#noABwUMUep)
  * [SeenThread](#no3oBo6LcT)


---
## <a name="noxTvQ00ZA" id="noxTvQ00ZA">PlaceholderTable</a>

### 概要(Summary)
SystemDictionary クラス内で使用される補助クラス (See: SystemDictionary).

SystemDictionary が現在ロード中のクラス情報を入れておくハッシュテーブル.


```cpp
    ((cite: hotspot/src/share/vm/classfile/placeholders.hpp))
    class PlaceholderTable : public TwoOopHashtable<Symbol*> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SystemDictionary クラスの _placeholders フィールド (static フィールド) に(のみ)格納されている.


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Hashtable holding placeholders for classes being loaded.
      static PlaceholderTable*       _placeholders;
```




### 詳細(Details)
See: [here](../doxygen/classPlaceholderTable.html) for details

---
## <a name="noABwUMUep" id="noABwUMUep">PlaceholderEntry</a>

### 概要(Summary)
PlaceholderTable クラス内で使用される補助クラス.

PlaceholderTable オブジェクト内に格納されるハッシュテーブル・エントリ.


```cpp
    ((cite: hotspot/src/share/vm/classfile/placeholders.hpp))
    // Placeholder objects represent classes currently being loaded.
    // All threads examining the placeholder table must hold the
    // SystemDictionary_lock, so we don't need special precautions
    // on store ordering here.
    // The system dictionary is the only user of this class.
    
    class PlaceholderEntry : public HashtableEntry<Symbol*> {
```




### 詳細(Details)
See: [here](../doxygen/classPlaceholderEntry.html) for details

---
## <a name="no3oBo6LcT" id="no3oBo6LcT">SeenThread</a>

### 概要(Summary)
PlaceholderEntry クラス用の補助クラス.

ある特定のクラスに対して, 現在ロードを行っているスレッド全員を表す.


```cpp
    ((cite: hotspot/src/share/vm/classfile/placeholders.hpp))
    // SeenThread objects represent list of threads that are
    // currently performing a load action on a class.
    // For class circularity, set before loading a superclass.
    // For bootclasssearchpath, set before calling load_instance_class.
    // Defining must be single threaded on a class/classloader basis
    // For DEFINE_CLASS, the head of the queue owns the
    // define token and the rest of the threads wait to return the
    // result the first thread gets.
    class SeenThread: public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classSeenThread.html) for details

---
