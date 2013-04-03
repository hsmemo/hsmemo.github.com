---
layout: default
title: SymbolTable クラス関連のクラス (TempNewSymbol, SymbolTable, StringTable, 及びそれらの補助クラス(StableMemoryChecker))
---
[Top](../index.html)

#### SymbolTable クラス関連のクラス (TempNewSymbol, SymbolTable, StringTable, 及びそれらの補助クラス(StableMemoryChecker))

これらは, Symbol オブジェクトやインターンされた文字列(Interned Strings) を管理するためのクラス (See: Symbol).


```
    ((cite: hotspot/src/share/vm/classfile/symbolTable.hpp))
    // The symbol table holds all Symbol*s and corresponding interned strings.
    // Symbol*s and literal strings should be canonicalized.
    //
    // The interned strings are created lazily.
    //
    // It is implemented as an open hash table with a fixed number of buckets.
    //
    // %note:
    //  - symbolTableEntrys are allocated in blocks to reduce the space overhead.
```


### クラス一覧(class list)

  * [SymbolTable](#no5HI7NMFi)
  * [StringTable](#noFSsVxz5N)
  * [TempNewSymbol](#nol16BHi53)
  * [StableMemoryChecker](#noLDRMO38C)


---
## <a name="no5HI7NMFi" id="no5HI7NMFi">SymbolTable</a>

### 概要(Summary)
全ての Symbol オブジェクトを管理するためのクラス
(なお, このクラスは CHeapObj クラスだが, public になっているのが static メソッドしかないため見た目は AllStatic に近い.
「Symbol を管理するための関数を納めた名前空間」といった感じ).

新しい Symbol を生成するファクトリメソッドや文字列から対応する Symbol を取得するメソッド等を提供している.


```
    ((cite: hotspot/src/share/vm/classfile/symbolTable.hpp))
    class SymbolTable : public Hashtable<Symbol*> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
SymbolTable クラスの _the_table フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* SymbolTable::create_table()
* SymbolTable::create_table(HashtableBucket* t, int length, int number_of_entries)




### 詳細(Details)
See: [here](../doxygen/classSymbolTable.html) for details

---
## <a name="noFSsVxz5N" id="noFSsVxz5N">StringTable</a>

### 概要(Summary)
全ての「インターンされた文字列オブジェクト(Interned Strings)」を管理するためのクラス
(なお, このクラスは CHeapObj クラスだが, public になっているのが static メソッドしかないため見た目は AllStatic に近い.
「インターンされた文字列を管理するための関数を納めた名前空間」といった感じ).

(インターンについては java.lang.String.intern() 参照)

文字列オブジェクト(java.lang.String) のインターン処理に関するメソッドを提供している.

(なお, ユーザーによる明示的な java.lang.String.intern() 以外でも StringTable クラスは使用されている.
 例えば, java.lang.Class.getName() の処理でクラス名を表す java.lang.String オブジェクトを返す場合には, 
 新しく作るのはもったいないので(?) intern されている文字列オブジェクトを返したりしている
 (See: JVM_GetClassName()))


```
    ((cite: hotspot/src/share/vm/classfile/symbolTable.hpp))
    class StringTable : public Hashtable<oop> {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
StringTable クラスの _the_table フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* StringTable::create_table()
* StringTable::create_table(HashtableBucket* t, int length, int number_of_entries)





### 詳細(Details)
See: [here](../doxygen/classStringTable.html) for details

---
## <a name="nol16BHi53" id="nol16BHi53">TempNewSymbol</a>

### 概要(Summary)
Symbol クラス用のユーティリティ・クラス.

ある Symbol オブジェクトの reference count を作業途中のあるスコープの中でだけ上げておきたい, 
という場合に使われる一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/classfile/symbolTable.hpp))
    // Class to hold a newly created or referenced Symbol* temporarily in scope.
    // new_symbol() and lookup() will create a Symbol* if not already in the
    // symbol table and add to the symbol's reference count.
    // probe() and lookup_only() will increment the refcount if symbol is found.
    class TempNewSymbol : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
実際に使用する際にはこんな感じになる.

* 右辺の Symbol 型の値が, 
  TempNewSymbol(Symbol *s) というコンストラクタで TempNewSymbol 型のオブジェクトに暗黙的にキャストされる. 
  そして, それが copy constructor に引き渡される模様.
 
* 右辺の方はこの場で TempNewSymbol のデストラクタが呼ばれて reference count が -1 され, 
  左辺の方は copy constructor で +1 されるので, この時点では reference count の値は変わらない.

* 最終的には, このスコープが終わって左辺の変数が死んだときにデストラクタで -1 される.


```
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
      TempNewSymbol class_name = SymbolTable::new_symbol(name, THREAD);
```

#### 使用箇所(where its instances are used)
HotSpot 内の様々な箇所で使用されている (#TODO).

(主に, 作業用に一時的に作った Symbol オブジェクトを作業終了時に自動的に解放する, といった用途で使われている模様)





### 詳細(Details)
See: [here](../doxygen/classTempNewSymbol.html) for details

---
## <a name="noLDRMO38C" id="noLDRMO38C">StableMemoryChecker</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).


```
    ((cite: hotspot/src/share/vm/classfile/symbolTable.cpp))
    #ifdef ASSERT
    class StableMemoryChecker : public StackObj {
```

### 使われ方(Usage)
StringTable::basic_add() 内で(のみ)使用されている.

(が, あまり意味がある使われ方になっていないような...?? (#TODO). 
コンストラクタ内で指定された文字列を内部にコピーし, verify() 時に同じものかどうかのチェックをするようだが,
コンストラクタ呼び出しはあっても verify() がないので実質的には使われていないように見える....)




### 詳細(Details)
See: [here](../doxygen/classStableMemoryChecker.html) for details

---
