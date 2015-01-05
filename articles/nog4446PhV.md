---
layout: default
title: methodOopDesc クラス関連のクラス (methodOopDesc, CompressedLineNumberWriteStream, CompressedLineNumberReadStream, BreakpointInfo, 及びそれらの補助クラス(SignatureTypePrinter))
---
[Top](../index.html)

#### methodOopDesc クラス関連のクラス (methodOopDesc, CompressedLineNumberWriteStream, CompressedLineNumberReadStream, BreakpointInfo, 及びそれらの補助クラス(SignatureTypePrinter))

これらは, Java レベルでの「メソッド」を表すためのクラス (See: [here](no7882m2Z.html) for details).


### クラス一覧(class list)

  * [methodOopDesc](#notZ98bLx_)
  * [CompressedLineNumberWriteStream](#noa5n48a_A)
  * [CompressedLineNumberReadStream](#no7m1LelTs)
  * [BreakpointInfo](#nofuYfNN7c)
  * [SignatureTypePrinter](#noJyovZ9I8)


---
## <a name="notZ98bLx_" id="notZ98bLx_">methodOopDesc</a>

### 概要(Summary)
Java レベルでの「メソッド」を表すためのクラス.
1つの methodOopDesc オブジェクトが 1つのメソッドに対応する.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    // A methodOop represents a Java method.
```


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    class methodOopDesc : public oopDesc {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
(#TODO)

* 各 instanceKlass オブジェクトの _methods フィールド内

#### 生成箇所(where its instances are created)
methodKlass::allocate() というファクトリメソッドが用意されており, その中で生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
* 
  ClassFileParser::parse_method()
  -&gt; oopFactory::new_method()
     -&gt; methodKlass::allocate()

* 
  methodOopDesc::make_invoke_method()
  -&gt; oopFactory::new_method()
     -&gt; (同上)
  
* 
  methodOopDesc::clone_with_new_data()
  -&gt; oopFactory::new_method()
     -&gt; (同上)

* 
  MethodHandleCompiler::get_method_oop
  -&gt; oopFactory::new_method()
     -&gt; (同上)
</pre></div>

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

* markOop  _mark  (下図中の header)

  oopDesc の共通ヘッダ (mark)

* wideKlassOop _metadata._klass (or narrowOop _metadata._compressed_klass)   (下図中の klass)

  oopDesc の共通ヘッダ (klass)

* constMethodOop    _constMethod   (下図中の constMethodOop)

  この methodOopDesc に対応する constMethodOopDesc オブジェクト.
  メソッド情報中の read-only なデータを格納する (See: constMethodOopDesc).

* constantPoolOop   _constants   (下図中の constants)

  この methodOopDesc が属するクラス用の constantPoolOopDesc オブジェクト.
  Constant Pool 情報を格納する (See: constantPoolOopDesc).

* methodDataOop     _method_data   (下図中の methodData)

  この methodOopDesc に対応する methodDataOopDesc オブジェクト.
  JIT コンパイラのための補助情報を格納する (See: methodDataOopDesc).

* int               _interpreter_invocation_count   (下図中の interp_invocation_count)

  JIT コンパイラ用のフィールド. インタープリタ状態で呼び出された回数を記録する.
  (なお, Tiered 時には...#TODO)

* AccessFlags       _access_flags   (下図中の access_flags)

  この methodOopDesc に対応する AccessFlags オブジェクト.
  アクセスフラグ情報(public, synchronized, etc)を管理する (See: AccessFlags).

* int               _vtable_index   (下図中の vtable_index)

  vtable 中におけるこのメソッドの番号(index).
  なお負値が入っている場合は index ではなく以下のような別の意味を持つ (See: VtableIndexFlag).


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
      // vtable index
      enum VtableIndexFlag {
        // Valid vtable indexes are non-negative (>= 0).
        // These few negative values are used as sentinels.
        highest_unused_vtable_index_value = -5,
        invalid_vtable_index    = -4,  // distinct from any valid vtable index
        garbage_vtable_index    = -3,  // not yet linked; no vtable layout yet
        nonvirtual_vtable_index = -2   // there is no need for vtable dispatch
        // 6330203 Note:  Do not use -1, which was overloaded with many meanings.
      };
```

* int               _result_index   (下図中の result_index)

  C++ Interpreter 用のフィールド (#ifdef CC_INTERP 時にしか定義されない).
  (#TODO).

* u2                _method_size   (下図中の method_size)

  この methodOopDesc の大きさ
  (わざわざフィールドが用意されているのは,
  ネイティブメソッドだとフィールドが追加されて大きさが変わるので単純な sizeof では取得できないため.
  下記の native_function や signature_handler 参照)

* u2                _max_stack   (下図中の max_stack)

  このメソッド実行中のオペランドスタックの最大の深さ
  (これはクラスファイル中に記載されている情報).

* u2                _max_locals   (下図中の max_locals)

  このメソッドが使用する局所変数の数
  (これはクラスファイル中に記載されている情報).

* u2                _size_of_parameters   (下図中の size_of_parameters)

  局所変数領域内でこのメソッドの引数(receiver + arguments)が占めるメモリ量 (単位は word).

* u1                _intrinsic_id   (下図中の intrinsic_id)

  このメソッドに対応する vmIntrinsics::ID の値 (See: vmIntrinsics).

* u2                _interpreter_throwout_count   (下図中の throwout_count)

  JIT コンパイラ用のフィールド. インタープリタ状態で例外が投げられた回数を記録する.

* u2                _number_of_breakpoints   (下図中の num_breakpoints)

  JVMTI の SetBreakpoint() でこのメソッド中に設定されたブレークポイントの数.
  (See: [here](no3718uV0.html) for details)

* InvocationCounter _invocation_counter   (下図中の invocation_counter)

  JIT コンパイラ使用時用のフィールド.
  このメソッドが呼び出された回数を管理する InvocationCounter オブジェクト (See: InvocationCounter).
  (See: [here](no2935G1h.html) for details)

* InvocationCounter _backedge_counter   (下図中の backedge_counter)

  JIT コンパイラ使用時用のフィールド.
  このメソッド内で実行された backward-branch の回数を管理する InvocationCounter オブジェクト (See: InvocationCounter).
  (See: [here](no2935G1h.html) and [here](no2935sgV.html) for details)

* jlong             _prev_time   (下図中の prev_time)

  Tiered Compilation 用のフィールド (#ifdef TIERED 時にしか定義されない).
  (#TODO).

* float             _rate   (下図中の rate)

  Tiered Compilation 用のフィールド (#ifdef TIERED 時にしか定義されない).
  (#TODO).

* int               _compiled_invocation_count   (下図中にはない)

  デバッグ用(開発時用)のフィールド (#ifndef PRODUCT 時にしか定義されない).
  JIT 生成コードが呼び出された回数を記録する.

* address _i2i_entry   (下図中の i2i)

  インタープリタ実行時にメソッドのエントリ部で行う処理をまとめたスタブコードへのポインタが格納されている
  (_from_interpreted_entry フィールドも参照).
  (See: [here](no3059n2f.html) for details)

* AdapterHandlerEntry* _adapter   (下図中の adapter)

  JIT コンパイラ使用時用のフィールド.
  インタープリタ実行のメソッドから JIT 生成コードが呼ばれる(あるいはその逆)場合のために,
  引数の marsharing 等を行うためのスタブコード (i2c_entry, c2i_entry) が格納されている.
  なお, このスタブはメソッドのリンク時に生成される
  (SharedRuntime::generate_i2c2i_adapters() 参照. (See: [here](no2935fWP.html) for details)).

* volatile address _from_compiled_entry   (下図中の from_compiled_entry)

  JIT コンパイラ使用時用のフィールド.
  このメソッドが JIT 生成コードから呼び出された場合のエントリポイントを指す
  (JIT 生成コードがあれば code, なければ c2i_entry を指している).

* nmethod* volatile _code   (下図中の code)

  JIT コンパイラ用のフィールド. JIT が生成したコードの先頭を指すポインタが格納される(まだ作ってなければ NULL).
  なお, ここのフィールドは JIT コンパイラが非同期に (NULLからnon-NULLに) 変更する.
  deopt 処理も (non-NULLからNULLに) 変更するが, こちらは safepoint 内で変更する.

* volatile address           _from_interpreted_entry   (下図中の from_interpreted_entry)

  このメソッドがインタープリタ実行のメソッドから呼び出された場合のエントリポイントを指す
  (JIT 生成コードがあれば code への i2c_entry, なければ i2i_entry を指している).

* (フィールドとしては定義されていない)   (下図中の native_function)

  フィールド宣言はされてないが, ネイティブメソッドの場合にはここにデータが存在する.
  格納されているのは, ネイティブ関数の先頭を指す関数ポインタ.
  (InterpreterGenerator::generate_native_entry() や SharedRuntime::generate_native_wrapper() 等を参照.
   (See: [here](no3059asZ.html) and [here](no293548G.html) for details))

* (フィールドとしては定義されていない)   (下図中の signature_handler)

  フィールド宣言はされてないが, ネイティブメソッドの場合にはここにデータが存在する.
  格納されているのは, 引数をネイティブのABI(レジスタ渡し等)に合わせて変換するための関数(signature handler).
  この関数はネイティブ関数に飛ぶ直前に呼び出される.
  (InterpreterGenerator::generate_native_entry() を参照. (See: [here](no3059asZ.html) for details))


なお, メモリ上でのレイアウトは以下の通り (ただしこれは 32bit 環境時であることに注意).

* 1列が1wordに相当. 1行に複数書かれているのは u2 や u1 の値が詰め込まれている箇所.
* GC 時の局所性向上のため, oop のフィールドと method_size を先頭にまとめて配置しているらしい
* インタープリタが使用するので, native_function と signature_handler のオフセットは固定でないとまずい, とのこと.
* メソッドは本当に大量に存在しているので, methodOop をコンパクトにまとめるのはメモリ使用量を下げる上でかなり重要, とのこと.
* なお, 以下に示すデータは methodOop から参照されている constMethodOop オブジェクト内に格納されている
  (コメントではそうではないような書き方だったが, 実際のコード上では constMethodOop 内にある).
  またバイトコード以外の項目は存在しない場合があり, 存在するかどうかは has_linenumber_table() 等のメソッドで確認できる
  (コメントでは access_flags にいれているみたいなことを書いていたが, constMethodOop 管理なのでコード上ではそうなってはいない).

    * 実際のバイトコード(メソッドのbody部)
    * line number table(line_number_table)情報 (を圧縮したもの)
    * checked exceptions table
    * local variable table(local_variable_table)情報


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    // Memory layout (each line represents a word). Note that most applications load thousands of methods,
    // so keeping the size of this structure small has a big impact on footprint.
    //
    // We put all oops and method_size first for better gc cache locality.
    //
    // The actual bytecodes are inlined after the end of the methodOopDesc struct.
    //
    // There are bits in the access_flags telling whether inlined tables are present.
    // Note that accessing the line number and local variable tables is not performance critical at all.
    // Accessing the checked exceptions table is used by reflection, so we put that last to make access
    // to it fast.
    //
    // The line number table is compressed and inlined following the byte codes. It is found as the first
    // byte following the byte codes. The checked exceptions table and the local variable table are inlined
    // after the line number table, and indexed from the end of the method. We do not compress the checked
    // exceptions table since the average length is less than 2, and do not bother to compress the local
    // variable table either since it is mostly absent.
    //
    // Note that native_function and signature_handler has to be at fixed offsets (required by the interpreter)
    //
    // |------------------------------------------------------|
    // | header                                               |
    // | klass                                                |
    // |------------------------------------------------------|
    // | constMethodOop                 (oop)                 |
    // | constants                      (oop)                 |
    // |------------------------------------------------------|
    // | methodData                     (oop)                 |
    // | interp_invocation_count                              |
    // |------------------------------------------------------|
    // | access_flags                                         |
    // | vtable_index                                         |
    // |------------------------------------------------------|
    // | result_index (C++ interpreter only)                  |
    // |------------------------------------------------------|
    // | method_size             | max_stack                  |
    // | max_locals              | size_of_parameters         |
    // |------------------------------------------------------|
    // | intrinsic_id, (unused)  |  throwout_count            |
    // |------------------------------------------------------|
    // | num_breakpoints         |  (unused)                  |
    // |------------------------------------------------------|
    // | invocation_counter                                   |
    // | backedge_counter                                     |
    // |------------------------------------------------------|
    // |           prev_time (tiered only, 64 bit wide)       |
    // |                                                      |
    // |------------------------------------------------------|
    // |                  rate (tiered)                       |
    // |------------------------------------------------------|
    // | code                           (pointer)             |
    // | i2i                            (pointer)             |
    // | adapter                        (pointer)             |
    // | from_compiled_entry            (pointer)             |
    // | from_interpreted_entry         (pointer)             |
    // |------------------------------------------------------|
    // | native_function       (present only if native)       |
    // | signature_handler     (present only if native)       |
    // |------------------------------------------------------|
```

### 備考(Notes)
なお, 実際の使用箇所では methodOop という別名(もしくはラッパークラス)で使われることが多い (See: methodOop).

### 備考(Notes)
methodOopDesc クラスは HotSpot が内部的に使用するデータ構造であり, Java のオブジェクトを表すものではない.

java.lang.reflect.Method オブジェクトや
java.lang.invoke.MethodHandle オブジェクトは instanceOopDesc で表現される.




### 詳細(Details)
See: [here](../doxygen/classmethodOopDesc.html) for details

---
## <a name="noa5n48a_A" id="noa5n48a_A">CompressedLineNumberWriteStream</a>

### 概要(Summary)
methodOopDesc クラス用の補助クラス.

methodOopDesc 内では line_number_tables 情報は大きいので圧縮して保持している. その圧縮処理を行うためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    // Utility class for compressing line number tables
    
    class CompressedLineNumberWriteStream: public CompressedWriteStream {
```




### 詳細(Details)
See: [here](../doxygen/classCompressedLineNumberWriteStream.html) for details

---
## <a name="no7m1LelTs" id="no7m1LelTs">CompressedLineNumberReadStream</a>

### 概要(Summary)
methodOopDesc クラス用の補助クラス.

methodOopDesc 内では line_number_tables 情報は大きいので圧縮して保持している.
その圧縮された情報を解凍するためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    // Utility class for decompressing line number tables
    
    class CompressedLineNumberReadStream: public CompressedReadStream {
```




### 詳細(Details)
See: [here](../doxygen/classCompressedLineNumberReadStream.html) for details

---
## <a name="nofuYfNN7c" id="nofuYfNN7c">BreakpointInfo</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能用のクラス).

JVMTI の SetBreakpoint() によって設定されたブレークポイントを表す.
1つのブレークポイントに付き1つの BreakpointInfo オブジェクトが生成される (See: [here](no3718uV0.html) for details).

なおコメントによると「もう少しフィールド等が増えてきたら methodOop.hpp ではなく独立したファイルに定義してもよい」とのこと.
また「現状では BreakpointInfo のリストに対して並行して変更する仕組みがないが
JVMTI の処理についてならとりあえずこれで問題ない.
なぜなら変更は safepoint 時だけであり, 読み込みは並行で行われるが safepoint 時以外だけなので」
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.hpp))
    /// Fast Breakpoints.
    
    // If this structure gets more complicated (because bpts get numerous),
    // move it into its own header.
    
    // There is presently no provision for concurrent access
    // to breakpoint lists, which is only OK for JVMTI because
    // breakpoints are written only at safepoints, and are read
    // concurrently only outside of safepoints.
    
    class BreakpointInfo : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 instanceKlass オブジェクトの _breakpoints フィールドに(のみ)格納されている.

(正確には, このフィールドは BreakpointInfo の線形リストを格納するフィールド.
BreakpointInfo オブジェクトは _next フィールドで次の BreakpointInfo オブジェクトを指せる構造になっている.
その instanceKlass オブジェクト内で生成した BreakpointInfo オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
methodOopDesc::set_breakpoint() 内で(のみ)生成されている (See: [here](no3718uV0.html) for details).




### 詳細(Details)
See: [here](../doxygen/classBreakpointInfo.html) for details

---
## <a name="noJyovZ9I8" id="noJyovZ9I8">SignatureTypePrinter</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか使用されない).

指定されたメソッドの引数や返値の型を outputStream に出力する機能を提供する.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.cpp))
    #ifndef PRODUCT
    class SignatureTypePrinter : public SignatureTypeNames {
```

### 使われ方(Usage)
methodOopDesc::print_name 内で(のみ)使用されている.

### 内部構造(Internal structure)
(スーパークラスである SignatureTypeNames クラスのメソッド以外で) 定義されているメソッドは以下の 2つだけ.
それぞれ, 引数の型, 返値の型を出力する.


```cpp
    ((cite: hotspot/src/share/vm/oops/methodOop.cpp))
      void print_parameters()              { _use_separator = false; iterate_parameters(); }
      void print_returntype()              { _use_separator = false; iterate_returntype(); }
```




### 詳細(Details)
See: [here](../doxygen/classSignatureTypePrinter.html) for details

---
