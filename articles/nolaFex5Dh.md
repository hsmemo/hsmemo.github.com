---
layout: default
title: ExceptionHandlerTable クラス関連のクラス (HandlerTableEntry, ExceptionHandlerTable, ImplicitExceptionTable)
---
[Top](../index.html)

#### ExceptionHandlerTable クラス関連のクラス (HandlerTableEntry, ExceptionHandlerTable, ImplicitExceptionTable)

これらは, JIT 生成コード内での例外ハンドラを管理するクラス (See: [here](no7882BaV.html) and [here](no7882OrP.html) for details).


### クラス一覧(class list)

  * [ExceptionHandlerTable](#noYC4XL_cC)
  * [HandlerTableEntry](#no_mOg2E7E)
  * [ImplicitExceptionTable](#not8RUVFpr)


---
## <a name="noYC4XL_cC" id="noYC4XL_cC">ExceptionHandlerTable</a>

### 概要(Summary)
JIT 生成コード内での例外ハンドラを表すクラス.

内部は HandlerTableEntry オブジェクトの1次元配列でできているが, データ構造は少し特殊な形になっている.

* 1次元配列に格納されている HandlerTableEntry オブジェクトは, 
  マシン語命令のアドレス(program counter)が同じもの同士でグルーピングされている.
  
* そして, 1次元配列全体としては以下のような感じになっている.

  { group1 の先頭, group1 の要素1, group1 の要素2, ..., group2 の先頭, group2 の要素1, ...}

* 各グループの先頭を表す HandlerTableEntry オブジェクトでは, 
  先頭の _bci フィールドに (bci ではなく) そのグループ内の HandlerTableEntry の個数が納められている.
  そして, その後にその個数分だけのグループ要素が並ぶ.
 

```cpp
    ((cite: hotspot/src/share/vm/code/exceptionHandlerTable.hpp))
    // An ExceptionHandlerTable is an abstraction over a list of subtables
    // of exception handlers for CatchNodes. Each subtable has a one-entry
    // header holding length and catch_pco of the subtable, followed
    // by 'length' entries for each exception handler that can be reached
    // from the corresponding CatchNode. The catch_pco is the pc offset of
    // the CatchNode in the corresponding nmethod. Empty subtables are dis-
    // carded.
    //
    // Structure of the table:
    //
    // table    = { subtable }.
    // subtable = header entry { entry }.
    // header   = a pair (number of subtable entries, catch pc offset, [unused])
    // entry    = a pair (handler bci, handler pc offset, scope depth)
    //
    // An ExceptionHandlerTable can be created from scratch, in which case
    // it is possible to add subtables. It can also be created from an
    // nmethod (for lookup purposes) in which case the table cannot be
    // modified.
    ...
    class ExceptionHandlerTable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Compile::FillExceptionTables() の中で要素が詰められている.

(なお, _handler_table というのが Compile クラスのオブジェクトが持つ ExceptionHandlerTable 型のフィールド)


```cpp
    ((cite: hotspot/src/share/vm/opto/output.cpp))
    void Compile::FillExceptionTables(uint cnt, uint *call_returns, uint *inct_starts, Label *blk_labels) {
    ...
        // Compute ExceptionHandlerTable subtable entry and add it
        // (skip empty blocks)
        if( n->is_Catch() ) {
    ...
    
          // Set the offset of the return from the call
          _handler_table.add_subtable(call_return, &handler_bcis, NULL, &handler_pcos);
```

#### 使用箇所(where its instances are used)
実際に例外が発生した際に, SharedRuntime::compute_compiled_exc_handler() 内で対応する例外ハンドラが探索されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    address SharedRuntime::compute_compiled_exc_handler(nmethod* nm, address ret_pc, Handle& exception,
                                                        bool force_unwind, bool top_frame_only) {
    ...
      // found handling method => lookup exception handler
      int catch_pco = ret_pc - nm->code_begin();
    
      ExceptionHandlerTable table(nm);
      HandlerTableEntry *t = table.entry_for(catch_pco, handler_bci, scope_depth);
      if (t == NULL && (nm->is_compiled_by_c1() || handler_bci != -1)) {
        // Allow abbreviated catch tables.  The idea is to allow a method
        // to materialize its exceptions without committing to the exact
        // routing of exceptions.  In particular this is needed for adding
        // a synthethic handler to unlock monitors when inlining
        // synchonized methods since the unlock path isn't represented in
        // the bytecodes.
        t = table.entry_for(catch_pco, -1, 0);
      }
```

探索処理の手順は以下のようになる.

1. まず, 先頭から見ていって pco(program counter offset) が等しい要素を見つける
   (ここがグループの先頭になる. この処理を行うのが subtable_for()).

2. 次に, その先頭要素から len() でグループの長さを取得し, 
   後はその長さ分だけ調べて handler_bci や scope_depth が一致する要素を見つけ出す
   (長さ分だけ調べて見つからなければ, 存在しないということになる).


```cpp
    ((cite: hotspot/src/share/vm/code/exceptionHandlerTable.cpp))
    HandlerTableEntry* ExceptionHandlerTable::entry_for(int catch_pco, int handler_bci, int scope_depth) const {
      HandlerTableEntry* t = subtable_for(catch_pco);
      if (t != NULL) {
        int l = t->len();
        while (l-- > 0) {
          t++;
          if (t->bci() == handler_bci && t->scope_depth() == scope_depth) return t;
        }
      }
      return NULL;
```



### 詳細(Details)
See: [here](../doxygen/classExceptionHandlerTable.html) for details

---
## <a name="no_mOg2E7E" id="no_mOg2E7E">HandlerTableEntry</a>

### 概要(Summary)
ExceptionHandlerTable 内に格納されるエントリ.

中身は (bci, pco) というペアになっており, 
nmethod 中での pc offset と元々の java コードにおける例外ハンドラ(の bytecode index) の対応を示している.


```cpp
    ((cite: hotspot/src/share/vm/code/exceptionHandlerTable.hpp))
    // A HandlerTableEntry describes an individual entry of a subtable
    // of ExceptionHandlerTable. An entry consists of a pair(bci, pco),
    // where bci is the exception handler bci, and pco is the pc offset
    // relative to the nmethod code start for the compiled exception
    // handler corresponding to the (interpreted) exception handler
    // starting at bci.
    //
    // The first HandlerTableEntry of each subtable holds the length
    // and catch_pco for the subtable (the length is the number of
    // subtable entries w/o header).
    
    class HandlerTableEntry {
```

(なお, 正確に言うと (bci, pco) というペアではなく (bci, pco, scope_depth) というタプル)


```cpp
    ((cite: hotspot/src/share/vm/code/exceptionHandlerTable.hpp))
      int _bci;
      int _pco;
      int _scope_depth;
```



### 詳細(Details)
See: [here](../doxygen/classHandlerTableEntry.html) for details

---
## <a name="not8RUVFpr" id="not8RUVFpr">ImplicitExceptionTable</a>

### 概要(Summary)
Implicit Null Exception 用の Exception Table.
NullPointerException が起こる pc と対応するハンドラの pc の対応を格納している.

コンパイル作業中に対応が記録されていき, 最終的に nmethod 内部に格納される.

nmethod 内部に格納された後は, 「先頭に table の要素数を示す word があり, 
その後に pc のペアが要素分だけ続く」という形になる.
(ただし要素がゼロの場合は, 先頭の要素数のフィールドも含めて, 一切領域は取られない)


```cpp
    ((cite: hotspot/src/share/vm/code/exceptionHandlerTable.hpp))
    // ----------------------------------------------------------------------------
    // Implicit null exception tables.  Maps an exception PC offset to a
    // continuation PC offset.  During construction it's a variable sized
    // array with a max size and current length.  When stored inside an
    // nmethod a zero length table takes no space.  This is detected by
    // nul_chk_table_size() == 0.  Otherwise the table has a length word
    // followed by pairs of <excp-offset, const-offset>.
    
    ...
    class ImplicitExceptionTable VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
コンパイル中に, Compile::Fill_buffer() の中で implicit null check の情報が収集される.


```cpp
    ((cite: hotspot/src/share/vm/opto/output.cpp))
    void Compile::Fill_buffer() {
    ...
            // If this is a null check, then add the start of the previous instruction to the list
            else if( mach->is_MachNullCheck() ) {
              inct_starts[inct_cnt++] = previous_offset;
            }
```

そして, Compile::FillExceptionTables() の中で要素が詰められている
(なお, _inc_table というのが Compile クラスのオブジェクトが持つ ImplicitExceptionTable 型のフィールド).


```cpp
    ((cite: hotspot/src/share/vm/opto/output.cpp))
    void Compile::FillExceptionTables(uint cnt, uint *call_returns, uint *inct_starts, Label *blk_labels) {
      _inc_table.set_size(cnt);
    ...
        // Handle implicit null exception table updates
        if( n->is_MachNullCheck() ) {
          uint block_num = b->non_connector_successor(0)->_pre_order;
          _inc_table.append( inct_starts[inct_cnt++], blk_labels[block_num].loc_pos() );
    ...
```

#### 使用箇所(where its instances are used)
nmethod::continuation_for_implicit_exception() の中で使用されている.

実際に exception が起こった pc を使って, テーブルから対応する offset を引き, 
それをコードのアドレスに直してリターンしている (対応するものがなければ NULL を返す).


```cpp
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    address nmethod::continuation_for_implicit_exception(address pc) {
      // Exception happened outside inline-cache check code => we are inside
      // an active nmethod => use cpc to determine a return address
      int exception_offset = pc - code_begin();
      int cont_offset = ImplicitExceptionTable(this).at( exception_offset );
    ...
      if (cont_offset == 0) {
        // Let the normal error handling report the exception
        return NULL;
      }
      return code_begin() + cont_offset;
```




### 詳細(Details)
See: [here](../doxygen/classImplicitExceptionTable.html) for details

---
