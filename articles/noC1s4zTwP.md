---
layout: default
title: VtableStubs クラス関連のクラス (VtableStub, VtableStubs)
---
[Top](../index.html)

#### VtableStubs クラス関連のクラス (VtableStub, VtableStubs)

これらは, CompiledIC 用の補助クラス
(See: CompiledIC).


### クラス一覧(class list)

  * [VtableStubs](#nofitS_8Y7)
  * [VtableStub](#noGWf2Cydp)


---
## <a name="nofitS_8Y7" id="nofitS_8Y7">VtableStubs</a>

### 概要(Summary)
CompiledIC が megamorphic になった際に使用する dynamic dispatch 用のスタブコード
(= vtable/itable 経由での呼び出しを行うコード)を生成するクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

生成したスタブは, vtable index と args_size(引数の数) が同じものについては流用しているため, 
(vtable index, args_size) というペアに1対1対応する.
また, 一度生成したスタブは破棄されることはない.


```
    ((cite: hotspot/src/share/vm/code/vtableStubs.hpp))
    // VtableStubs creates the code stubs for compiled calls through vtables.
    // There is one stub per (vtable index, args_size) pair, and the stubs are
    // never deallocated. They don't need to be GCed because they contain no oops.
    
    class VtableStubs : AllStatic {
```

### 使われ方(Usage)
itable 経由での呼び出しコード(=インターフェースメソッド呼びコード)を生成する create_itable_stub() と, 
vtable 経由での呼び出しコード(=クラスメソッド呼びコード)を生成する create_vtable_stub() というメソッドを持つ.


```
    ((cite: hotspot/src/share/vm/code/vtableStubs.hpp))
      static VtableStub* create_vtable_stub(int vtable_index);
      static VtableStub* create_itable_stub(int vtable_index);
```

### 内部構造(Internal structure)
内部では, 生成したスタブコード 1つ1つを VtableStub オブジェクトとして管理している.

生成した VtableStub オブジェクトは, ハッシュに格納して管理する.


```
    ((cite: hotspot/src/share/vm/code/vtableStubs.hpp))
      static VtableStub* _table[N];                  // table of existing stubs
      static int         _number_of_vtable_stubs;    // number of stubs created so far (for statistics)
```

ハッシュの lookup 処理, 及び新しい要素の追加処理は, 以下の通り (普通の chain hash).


```
    ((cite: hotspot/src/share/vm/code/vtableStubs.cpp))
    VtableStub* VtableStubs::lookup(bool is_vtable_stub, int vtable_index) {
      MutexLocker ml(VtableStubs_lock);
      unsigned hash = VtableStubs::hash(is_vtable_stub, vtable_index);
      VtableStub* s = _table[hash];
      while( s && !s->matches(is_vtable_stub, vtable_index)) s = s->next();
      return s;
    }
    
    
    void VtableStubs::enter(bool is_vtable_stub, int vtable_index, VtableStub* s) {
      MutexLocker ml(VtableStubs_lock);
      assert(s->matches(is_vtable_stub, vtable_index), "bad vtable stub");
      unsigned int h = VtableStubs::hash(is_vtable_stub, vtable_index);
      // enter s at the beginning of the corresponding list
      s->set_next(_table[h]);
      _table[h] = s;
      _number_of_vtable_stubs++;
    }
```




### 詳細(Details)
See: [here](../doxygen/classVtableStubs.html) for details

---
## <a name="noGWf2Cydp" id="noGWf2Cydp">VtableStub</a>

### 概要(Summary)
dynamic dispatch 用のスタブコードを格納するためのクラス. 1つの VtableStub オブジェクトが 1つのスタブに対応する.

VtableStubs クラスによって生成され, VtableStubs クラス内のハッシュに格納されて管理される.


```
    ((cite: hotspot/src/share/vm/code/vtableStubs.hpp))
    // A VtableStub holds an individual code stub for a pair (vtable index, #args) for either itables or vtables
    // There's a one-to-one relationship between a VtableStub and such a pair.
    
    class VtableStub {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
CompiledIC::set_to_megamorphic() から呼び出される VtableStubs::create_stub() で生成される.

(より正確には, その中で vtable か itable かに応じて
VtableStubs::create_vtable_stub() または VtableStubs::create_itable_stub() が呼び出され,
そこで生成される)




### 詳細(Details)
See: [here](../doxygen/classVtableStub.html) for details

---
