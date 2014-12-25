---
layout: default
title: instanceKlass の vtable/itable 関連のクラス (klassVtable, vtableEntry, itableOffsetEntry, itableMethodEntry, klassItable, 及びそれらの補助クラス(InterfaceVisiterClosure, CountInterfacesClosure, SetupItableClosure, VtableStats))
---
[Top](../index.html)

#### instanceKlass の vtable/itable 関連のクラス (klassVtable, vtableEntry, itableOffsetEntry, itableMethodEntry, klassItable, 及びそれらの補助クラス(InterfaceVisiterClosure, CountInterfacesClosure, SetupItableClosure, VtableStats))

これらは, invokevirtual 及び invokeinterface 命令でのダイナミックディスパッチを実現するためのクラス (See: [here](no3059kgY.html) for details).

### 概要(Summary)
ダイナミックディスパッチ用のテーブルは各 instanceKlass オブジェクト内に格納されている.

vtable および itable と呼ばれており, それぞれ invokevirtual および invokeinterface 用
(See: instanceKlass).


```cpp
    ((cite: hotspot/src/share/vm/oops/instanceKlass.hpp))
    class instanceKlass: public Klass {
    ...
      // embedded Java vtable follows here
      // embedded Java itables follows here
```

それぞれの内部構造は以下の通り.

* vtable の内部構造
  
  vtableEntry オブジェクトの配列になっている.

* itable の内部構造
  
  itableOffsetEntry オブジェクトと itableMethodEntry オブジェクトという 2種類のオブジェクトで構成されている (備考も参照).

  まず適切な vtable を選択するための offset table が格納されている
  (より具体的には itableOffsetEntry オブジェクトの配列が埋め込まれている).
  
  そしてその直後に, それぞれの interface 用の vtable が格納されている
  (より具体的には itableMethodEntry オブジェクトの配列が埋め込まれている).
  
  (なお, 一番先頭の itableOffsetEntry だけは offset table の長さ情報が入っている ?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    // Format of an itable
    //
    //    ---- offset table ---
    //    klassOop of interface 1             \
    //    offset to vtable from start of oop  / offset table entry
    //    ...
    //    klassOop of interface n             \
    //    offset to vtable from start of oop  / offset table entry
    //    --- vtable for interface 1 ---
    //    methodOop                           \
    //    compiler entry point                / method table entry
    //    ...
    //    methodOop                           \
    //    compiler entry point                / method table entry
    //    -- vtable for interface 2 ---
    //    ...
```


また instanceKlass オブジェクト内の vtable や itable へのアクセスを容易化するために, klassVtable 及び klassItable という補助クラスも用意されている.
これらはエントリへのアクセサ関数を定義しただけのクラス.

(アクセス用の klassVtable オブジェクト/klassItable オブジェクトは, instanceKlass::vtable() メソッド及び instanceKlass::itable() メソッドで取得できる.
 あるいは, method_at_vtable(int index)/method_at_itable(klassOop holder, int index, TRAPS) で直接 methodOop を取得することもできる模様)


### 備考(Notes)
invokevirtual は単一継承なので, あるクラスを継承した全てのサブクラスにおいて,
メソッド名と vtable index の対応は一意にできる.

逆に, invokeinterface の場合は, そのインターフェースを implements している全てのクラスに対して
メソッド名と vtable index を一意に対応づけるのは (各クラスが1つしかvtableを持てないならば) かなり難しい.

そこで各クラスは, invokevirtual 用の vtable とは別に,
implement しているインターフェース1つ1つについてそれぞれ vtable を持っている.
そして, invokeinterface の場合は, まずどの interface として使われたかという情報から適切な vtable を選択し,
その後に ConstantPoolCacheEntry から取得した index に基づいて vtable 中の methodOop を選択する.


(実際にテーブルから取り出す作業は instanceKlass::method_at_itable() を見ると分かりやすい.
offset table を先頭から iterate していって指定の interface klassOop と一致するエントリを探し,
そこから offset を取り出して, vtable を取得して..., という流れ.
なお対応する itableOffsetEntry が見つからなければ java_lang_IncompatibleClassChangeError,
また vtable に入っていたエントリが NULL なら java_lang_AbstractMethodError になる模様)



### クラス一覧(class list)

  * [vtableEntry](#noyQxj-P6s)
  * [itableOffsetEntry](#noP1cC-HAT)
  * [itableMethodEntry](#noN7L6iOfj)
  * [klassVtable](#noinRBqyrl)
  * [klassItable](#nocPdFozxl)
  * [InterfaceVisiterClosure](#no8w02VroZ)
  * [CountInterfacesClosure](#no0fqX0YYN)
  * [SetupItableClosure](#nowCyVb9zc)
  * [VtableStats](#noab45njCp)


---
## <a name="noyQxj-P6s" id="noyQxj-P6s">vtableEntry</a>

### 概要(Summary)
vtable 内の要素を表すクラス.
1つの vtableEntry が 1つの要素に対応する (See: [here](no3059kgY.html) for details).

(また, vtable に関連した定数値もこのクラスに定義されている) 

vtable は vtableEntry の配列として構成され, instanceKlass 内に埋め込まれている.

(なお, 実際には vtableEntry オブジェクトは methodOopDesc へのポインタだけからなるデータ構造になっている.
このため vtable 自体は methodOopDesc へのポインタを並べた配列になっている)


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    // private helper class for klassVtable
    // description of entry points:
    //    destination is interpreted:
    //      from_compiled_code_entry_point -> c2iadapter
    //      from_interpreter_entry_point   -> interpreter entry point
    //    destination is compiled:
    //      from_compiled_code_entry_point -> nmethod entry point
    //      from_interpreter_entry_point   -> i2cadapter
    class vtableEntry VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を保持する.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
      methodOop _method;
```




### 詳細(Details)
See: [here](../doxygen/classvtableEntry.html) for details

---
## <a name="noP1cC-HAT" id="noP1cC-HAT">itableOffsetEntry</a>

### 概要(Summary)
itable 内の要素を表すクラス (See: [here](no3059kgY.html) for details).

itable 内には, まず適切な vtable を選択するための offset table が格納されている.
そしてその直後にそれぞれの interface 用の vtable が格納されている.

itableOffsetEntry は offset table 内の要素を表す.
1つの itableOffsetEntry が 1つの要素に対応する.

offset table は itableOffsetEntry の配列として構成され, instanceKlass 内に埋め込まれている.

(また, itable に関連した定数値もこのクラスに定義されている)


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    class itableOffsetEntry VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classitableOffsetEntry.html) for details

---
## <a name="noN7L6iOfj" id="noN7L6iOfj">itableMethodEntry</a>

### 概要(Summary)
itable 内の要素を表すクラス (See: [here](no3059kgY.html) for details).

itable 内には, まず適切な vtable を選択するための offset table が格納されている.
そしてその直後にそれぞれの interface 用の vtable が格納されている.

itableMethodEntry はそれぞれの interface 用の vtable 内の要素を表す.
1つの itableMethodEntry が 1つの要素に対応する.

それぞれの interface 用の vtable は itableMethodEntry の配列として構成され, instanceKlass 内に埋め込まれている.

(なお, 実際には itableMethodEntry オブジェクトは methodOopDesc へのポインタだけからなるデータ構造になっている.
このためそれぞれの interface 用の vtable 自体は methodOopDesc へのポインタを並べた配列になっている)

(また, itable に関連した定数値もこのクラスに定義されている)


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    class itableMethodEntry VALUE_OBJ_CLASS_SPEC {
```




### 詳細(Details)
See: [here](../doxygen/classitableMethodEntry.html) for details

---
## <a name="noinRBqyrl" id="noinRBqyrl">klassVtable</a>

### 概要(Summary)
invokevirtual 用の vtable の処理で使用される一時オブジェクト(ResourceObjクラス).

instanceKlass オブジェクト内に埋め込まれている vtable 情報にアクセスするためのクラス (See: [here](no3059kgY.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    // A klassVtable abstracts the variable-length vtable that is embedded in instanceKlass
    // and arrayKlass.  klassVtable objects are used just as convenient transient accessors to the vtable,
    // not to actually hold the vtable data.
    // Note: the klassVtable should not be accessed before the class has been verified
    // (until that point, the vtable is uninitialized).
    
    // Currently a klassVtable contains a direct reference to the vtable data, and is therefore
    // not preserved across GCs.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    class klassVtable : public ResourceObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
instanceKlass::vtable() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

#### 使用箇所(where its instances are used)
vtable の初期化処理や vtable 内のポインタの GC 処理で使用されている模様
(See: klassVtable::initialize_vtable(), klassVtable::oop_oop_iterate(), ).

(なお, 肝心の invokevirtual のダイナミックディスパッチ処理時には,
klassVtable は使わずに直接 vtable 内のデータにアクセスしている)




### 詳細(Details)
See: [here](../doxygen/classklassVtable.html) for details

---
## <a name="nocPdFozxl" id="nocPdFozxl">klassItable</a>

### 概要(Summary)
invokeinterface 用の vtable (= itable) の処理で使用される一時オブジェクト(ResourceObjクラス).

instanceKlass オブジェクト内に埋め込まれている itable 情報にアクセスするためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.hpp))
    class klassItable : public ResourceObj {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
instanceKlass::itable() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

#### 使用箇所(where its instances are used)
itable の初期化処理や itable 内のポインタの GC 処理で使用されている模様
(See: klassItable::initialize_itable(), klassItable::oop_oop_iterate(), ).

(なお, 肝心の invokeinterface のダイナミックディスパッチ処理時には,
klassItable は使わずに直接 itable 内のデータにアクセスしている)




### 詳細(Details)
See: [here](../doxygen/classklassItable.html) for details

---
## <a name="no8w02VroZ" id="no8w02VroZ">InterfaceVisiterClosure</a>

### 概要(Summary)
itable 内のエントリに対して何らかの処理を行う Closure クラス (の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.cpp))
    // Setup
    class InterfaceVisiterClosure : public StackObj {
```

### 使われ方(Usage)
itable 内のエントリを処理する doit() メソッドを備えている.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.cpp))
      virtual void doit(klassOop intf, int method_count) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classInterfaceVisiterClosure.html) for details

---
## <a name="no0fqX0YYN" id="no0fqX0YYN">CountInterfacesClosure</a>

### 概要(Summary)
klassItable クラス内で使用される補助クラス.

必要な itableMethodEntry 及び itableOffsetEntry の個数を数える Closure.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.cpp))
    class CountInterfacesClosure : public InterfaceVisiterClosure {
```

### 使われ方(Usage)
klassItable::compute_itable_size() 内, 及び
klassItable::setup_itable_offset_table() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classCountInterfacesClosure.html) for details

---
## <a name="nowCyVb9zc" id="nowCyVb9zc">SetupItableClosure</a>

### 概要(Summary)
klassItable クラス内で使用される補助クラス.

各 itableOffsetEntry に対して itableOffsetEntry::initialize() による初期化を行う Closure.


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.cpp))
    class SetupItableClosure : public InterfaceVisiterClosure  {
```

### 使われ方(Usage)
klassItable::setup_itable_offset_table() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classSetupItableClosure.html) for details

---
## <a name="noab45njCp" id="noab45njCp">VtableStats</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

vtable の統計情報収集用の関数を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/oops/klassVtable.cpp))
    class VtableStats : AllStatic {
```

### 使われ方(Usage)
klassVtable::print_statistics() 内で(のみ)使用されている.
そして, この関数は print_statistics() 内で(のみ)呼び出されている.

(ただし, このクラスは (デバッグ時であることに加えて) PrintVtableStats オプションが指定されている場合にしか使用されない)


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    #ifndef PRODUCT
    ...
    void print_statistics() {
    ...
      if (PrintVtableStats) {
        klassVtable::print_statistics();
```




### 詳細(Details)
See: [here](../doxygen/classVtableStats.html) for details

---
