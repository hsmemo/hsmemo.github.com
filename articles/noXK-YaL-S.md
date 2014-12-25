---
layout: default
title: HashTable クラス関連のクラス (BasicHashtableEntry, HashtableEntry, HashtableBucket, BasicHashtable, Hashtable, TwoOopHashtable)
---
[Top](../index.html)

#### HashTable クラス関連のクラス (BasicHashtableEntry, HashtableEntry, HashtableBucket, BasicHashtable, Hashtable, TwoOopHashtable)

これらは, 「ハッシュテーブル」として働くユーティリティ・クラス.

特に symbol table や string table としての使用を想定している, とのこと (See: SymbolTable, StringTable).

なお, 内部実装としては open hash で bucket 数は固定.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    // This is a generic hashtable, designed to be used for the symbol
    // and string tables.
    //
    // It is implemented as an open hash table with a fixed number of buckets.
    //
    // %note:
    //  - TableEntrys are allocated in blocks to reduce the space overhead.
```

### 概要(Summary)
このハッシュテーブルは, 内部的には以下の3つのクラスから構成されている.

  * HashtableEntry :
    ハッシュテーブル内の1要素を表すクラス
  * HashtableBucket :
    ハッシュテーブル内の bucket を表すクラス. 1つの bucket 内には複数の HashtableEntry が線形リスト状になって格納されている.
  * Hashtable :
    ハッシュテーブル全体を表すクラス. 複数の HashtableBucket からなる.

より正確には, 以下のようなクラス階層を持つ.

  * BasicHashtableEntry -- ハッシュテーブル内の要素を表すクラスの基底クラス.
    * HashtableEntry    -- ハッシュテーブル内の要素を表すクラス

  * HashtableBucket     -- ハッシュテーブル内の bucket を表すクラス

  * BasicHashtable      -- Hashtable を表すクラスの基底クラス.
    * Hashtable         -- ハッシュテーブルを表すクラス.
      * TwoOopHashtable -- Hashtable クラスの変種.



### クラス一覧(class list)

  * [BasicHashtableEntry](#noWSwnm6NH)
  * [HashtableEntry](#noIbguk23X)
  * [HashtableBucket](#noo9eEQOkF)
  * [BasicHashtable](#nopcvK5apg)
  * [Hashtable](#noQnwxCyro)
  * [TwoOopHashtable](#noRR0w6iAv)


---
## <a name="noWSwnm6NH" id="noWSwnm6NH">BasicHashtableEntry</a>

### 概要(Summary)
HashTable 内の要素を表すクラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    class BasicHashtableEntry : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.

* _hash フィールド : ハッシュ値を格納する
* _next フィールド : bucket 中での次の要素を示す


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
      unsigned int         _hash;           // 32-bit hash for item
    
      // Link to next element in the linked list for this bucket.  EXCEPT
      // bit 0 set indicates that this entry is shared and must not be
      // unlinked from the table. Bit 0 is set during the dumping of the
      // archive. Since shared entries are immutable, _next fields in the
      // shared entries will not change.  New entries will always be
      // unshared and since pointers are align, bit 0 will always remain 0
      // with no extra effort.
      BasicHashtableEntry* _next;
```




### 詳細(Details)
See: [here](../doxygen/classBasicHashtableEntry.html) for details

---
## <a name="noIbguk23X" id="noIbguk23X">HashtableEntry</a>

### 概要(Summary)
BasicHashtableEntry クラスの具象サブクラスの1つ.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    template <class T> class HashtableEntry : public BasicHashtableEntry {
```

### 内部構造(Internal structure)
BasicHashtableEntry のフィールドに加えて, 
ハッシュ値の計算の基となった値自体を格納するための 
_literal フィールドが追加されている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
      T               _literal;          // ref to item in table.
```




### 詳細(Details)
See: [here](../doxygen/classHashtableEntry.html) for details

---
## <a name="noo9eEQOkF" id="noo9eEQOkF">HashtableBucket</a>

### 概要(Summary)
HashTable クラス内で使用される補助クラス.

ハッシュテーブル内の bucket を表すクラス.
中身は BasicHashtableEntry オブジェクトを格納するための線形リストになっており, HashTable クラスはこのクラスを用いてハッシュテーブルを構築する.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    class HashtableBucket : public CHeapObj {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む.

(BasicHashtableEntry は _next フィールドを使って線形リストにできるので, 
実際にはこのフィールドに BasicHashtableEntry のリストが格納されている.
(See: BasicHashtable::add_entry()))


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
      BasicHashtableEntry*       _entry;
```




### 詳細(Details)
See: [here](../doxygen/classHashtableBucket.html) for details

---
## <a name="nopcvK5apg" id="nopcvK5apg">BasicHashtable</a>

### 概要(Summary)
ハッシュテーブルとして働くクラスの基底クラス.

(実質的には, Hashtable クラス用の補助クラスといった感じ)


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    class BasicHashtable : public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 内部構造(Internal structure)
内部には HashtableBucket オブジェクトの配列を保持している.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
      HashtableBucket*  _buckets;
```




### 詳細(Details)
See: [here](../doxygen/classBasicHashtable.html) for details

---
## <a name="noQnwxCyro" id="noQnwxCyro">Hashtable</a>

### 概要(Summary)
BasicHashtable クラスの具象サブクラスの1つ.


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    template <class T> class Hashtable : public BasicHashtable {
```




### 詳細(Details)
See: [here](../doxygen/classHashtable.html) for details

---
## <a name="noRR0w6iAv" id="noRR0w6iAv">TwoOopHashtable</a>

### 概要(Summary)
特殊な HashTable クラス.

普通の HashTable クラスと異なり, ハッシュ値の計算に値を2個使用する
(より具体的に言うと, compute_hash() メソッドが １引数ではなく２引数になっている.
これは, Symbol* 型の引数だけではなくクラスローダーを表す Handle 型の引数まで増えているため.)


```cpp
    ((cite: hotspot/src/share/vm/utilities/hashtable.hpp))
    //  Verions of hashtable where two handles are used to compute the index.
    
    template <class T> class TwoOopHashtable : public Hashtable<T> {
```




### 詳細(Details)
See: [here](../doxygen/classTwoOopHashtable.html) for details

---
