---
layout: default
title: OopMap クラス関連のクラス (OopMapValue, OopMap, OopMapSet, OopMapStream, DerivedPointerTable, DerivedPointerTableDeactivate, 及びそれらの補助クラス(DoNothingClosure, DerivedPointerEntry))
---
[Top](../index.html)

#### OopMap クラス関連のクラス (OopMapValue, OopMap, OopMapSet, OopMapStream, DerivedPointerTable, DerivedPointerTableDeactivate, 及びそれらの補助クラス(DoNothingClosure, DerivedPointerEntry))

#Under Construction

これらは, JIT コンパイラが GC 処理と協調するために生成するメタ情報(OopMap)を表すクラス.

### 概要(Summary)
JIT 生成コードの実行中にも GC (及びその他の safepoint 処理) が発生する可能性はあるため, 
「safepoint 停止した時点でどこにポインタ値が存在しているか」という情報(frame map または OopMap)がないといけない.

JIT 生成コードの場合, OopMap 情報は JIT コンパイラが生成する.

OopMap クラスは, ある safepoint 地点での「ポインタ値が存在する位置全体」の情報を表す.
OopMap 内には OopMapValue というエントリが格納されており, OopMapValue 1つが「レジスタと種別のペア」1つに対応する.

その他に, 複数の OopMap をまとめて扱うための OopMapSet という集合クラスもある.

また, OopMap に対する繰り返し処理用に OopMapStream というイテレータオブジェクトが用意されている.

### 備考(Notes)
実際には, 「ポインタ値かどうか」だけではなく, 以下のような情報を記録している.

* 生きているのか死んでいるのか? 
* 生きているなら oop か narrow_oop か value(non-oop) か? 
* 何かのポインタから計算したポインタ(derived pointer)かどうか? 
* 何かの値をスタック等に退避したもの(callee saved)かどうか? 
  (callee saved の場合は, ポインタかどうかが呼び出し元次第で変わる)
    
(なお, callee saved の場合は
「何という stack slot (VMReg 番号) が callee saved で, 対応する元の register はどれか(VMReg 番号は何か)」 
 という情報を格納する模様.
 RegisterSaver::save_live_registers() 中の set_callee_saved() の利用箇所を参照)


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    // Interface for generating the frame map for compiled code.  A frame map
    // describes for a specific pc whether each register and frame stack slot is:
    //   Oop         - A GC root for current frame
    //   Value       - Live non-oop, non-float value: int, either half of double
    //   Dead        - Dead; can be Zapped for debugging
    //   CalleeXX    - Callee saved; also describes which caller register is saved
    //   DerivedXX   - A derived oop; original oop is described.
    //
    // OopMapValue describes a single OopMap entry
```

なお, 上のコメントでは 5種類と書かれているが, 実際には narrow_oop という種別もあるため 6種類.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
      enum oop_types {              // must fit in type_bits
             unused_value =0,       // powers of 2, for masking OopMapStream
             oop_value = 1,
             value_value = 2,
             narrowoop_value = 4,
             callee_saved_value = 8,
             derived_oop_value= 16 };
```



### クラス一覧(class list)

  * [OopMap](#noTAHZHCXB)
  * [OopMapValue](#no3uQ4wcvj)
  * [OopMapStream](#nodanA-Spe)
  * [OopMapSet](#noYXYaU_ex)
  * [DerivedPointerTable](#no0SCG9_8P)
  * [DerivedPointerTableDeactivate](#nokteatfkf)
  * [DoNothingClosure](#no2lY71wwv)
  * [DerivedPointerEntry](#no9dRUmddP)


---
## <a name="noTAHZHCXB" id="noTAHZHCXB">OopMap</a>

### 概要(Summary)
JIT Compiler による OopMap 処理用のユーティリティ・クラス.
OopMap 情報を作成する処理で使用される一時オブジェクト(ResourceObjクラス).

作成される OopMap 情報を表す.
Safepoint に指定された実行地点毎に 1つの OopMap オブジェクトが存在する.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    class OopMap: public ResourceObj {
```

### 内部構造(Internal structure)
内部には複数の OopMapValue オブジェクトを格納している.
なお, メモリ消費量を抑えるため CompressedWriteStream によって圧縮した形で格納している模様.




### 詳細(Details)
See: [here](../doxygen/classOopMap.html) for details

---
## <a name="no3uQ4wcvj" id="no3uQ4wcvj">OopMapValue</a>

### 概要(Summary)
OopMap クラス用の補助クラス.

OopMap オブジェクト内に格納されるエントリ.
1つの OopMapValue オブジェクトが「レジスタと種別のペア」1つに対応する.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    class OopMapValue: public StackObj {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の 2つのみ.

* short _value
* short _content_reg
  
(なお, OopMapValue 内には, レジスタ番号(VMReg番号) と oopmap 種別(oop_types) のペアを格納されている.
さらに callee saved や derived pointer の場合には補足用の情報も格納されている)

_value フィールドに VMReg と oop_types のペアの情報が格納されている.
(short は16bitあるので積めれば両方を1つのshortに入れられる模様)

_content_reg フィールドに callee saved や derived pointer 用の補足情報を格納している 
(関連する VMReg 番号を格納している模様).


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
      short _value;
```


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
      short _content_reg;
```




### 詳細(Details)
See: [here](../doxygen/classOopMapValue.html) for details

---
## <a name="nodanA-Spe" id="nodanA-Spe">OopMapStream</a>

### 概要(Summary)
OopMap クラス用のユーティリティ・クラス.

OopMap オブジェクト内にある OopMapValue オブジェクトをたどるためのイテレータクラス(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    class OopMapStream : public StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classOopMapStream.html) for details

---
## <a name="noYXYaU_ex" id="noYXYaU_ex">OopMapSet</a>

### 概要(Summary)
OopMap クラス用のユーティリティ・クラス.

複数の OopMap オブジェクトを格納できるコンテナクラス. (一つのメソッド内の全部の OopMap をまとめるために使われている? #TODO)


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    class OopMapSet : public ResourceObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
* add_gc_map() で OopMap を追加できる模様. (stub 内など, 時々明示的に add_gc_map() している箇所がある) (#TODO)

* また, find_map_at_offset() で指定した PC に対応する OopMap を取得できる模様. (#TODO)

* また, all_do() で stack walk 処理をやっている模様. (#TODO)




### 詳細(Details)
See: [here](../doxygen/classOopMapSet.html) for details

---
## <a name="no0SCG9_8P" id="no0SCG9_8P">DerivedPointerTable</a>

### 概要(Summary)
C2 JIT Compiler 用の補助クラス (#ifdef COMPILER2 時にしか定義されない).

derived pointer 情報を入れるテーブルを納めた名前空間(AllStatic クラス).
GC 中に見つかった derived pointer 情報をこの中に溜めていき, GC 後に base pointer のアドレスに基づいて値を更新するために使う.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    // Derived pointer support. This table keeps track of all derived points on a
    // stack.  It is cleared before each scavenge/GC.  During the traversal of all
    // oops, it is filled in with references to all locations that contains a
    // derived oop (assumed to be very few).  When the GC is complete, the derived
    // pointers are updated based on their base pointers new value and an offset.
    #ifdef COMPILER2
    class DerivedPointerTable : public AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. GC の開始前に DerivedPointerTable::clear() を呼び出して, 中身を初期化しておく.
2. GC 中に derived pointer を見つけたら, DerivedPointerTable::add() で登録しておく.
3. GC が終わった時に DerivedPointerTable::update_pointers() を呼ぶと, 登録しておいた derived pointer の値が書き換わる.


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
      static void clear();                       // Called before scavenge/GC
      static void add(oop *derived, oop *base);  // Called during scavenge/GC
      static void update_pointers();             // Called after  scavenge/GC
```




### 詳細(Details)
See: [here](../doxygen/classDerivedPointerTable.html) for details

---
## <a name="nokteatfkf" id="nokteatfkf">DerivedPointerTableDeactivate</a>

### 概要(Summary)
C2 JIT Compiler 用の補助クラス (#ifdef COMPILER2 時にしか定義されない).

ソースコード中のあるスコープの間だけ, DerivedPointerTable を "deactivate" するための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/compiler/oopMap.hpp))
    // A utility class to temporarily "deactivate" the DerivedPointerTable.
    // (Note: clients are responsible for any MT-safety issues)
    class DerivedPointerTableDeactivate: public StackObj {
```

### 内部構造(Internal structure)
コンストラクタで DerivedPointerTable::set_active(false) が呼び出され, デストラクタで元の値に戻される.




### 詳細(Details)
See: [here](../doxygen/classDerivedPointerTableDeactivate.html) for details

---
## <a name="no2lY71wwv" id="no2lY71wwv">DoNothingClosure</a>

### 概要(Summary)
(#Under Construction)




### 詳細(Details)
See: [here](../doxygen/classDoNothingClosure.html) for details

---
## <a name="no9dRUmddP" id="no9dRUmddP">DerivedPointerEntry</a>

### 概要(Summary)
(#Under Construction)




### 詳細(Details)
See: [here](../doxygen/classDerivedPointerEntry.html) for details

---
