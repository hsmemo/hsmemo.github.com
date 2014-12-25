---
layout: default
title: SharedRuntime クラス関連のクラス (SharedRuntime, AdapterHandlerEntry, AdapterHandlerLibrary, 及びそれらの補助クラス(MethodArityHistogram, AdapterFingerPrint, AdapterHandlerTable, AdapterHandlerTableIterator))
---
[Top](../index.html)

#### SharedRuntime クラス関連のクラス (SharedRuntime, AdapterHandlerEntry, AdapterHandlerLibrary, 及びそれらの補助クラス(MethodArityHistogram, AdapterFingerPrint, AdapterHandlerTable, AdapterHandlerTableIterator))

これらは, Java プログラムの実行を補佐するクラス.
より具体的に言うと, アセンブリでは書くことが難しい複雑な機能を提供するクラス(Runtimeクラス), 及び JIT Compiler 用のスタブコードを生成するクラス.


### クラス一覧(class list)

  * [SharedRuntime](#noQwiO5QYM)
  * [AdapterHandlerLibrary](#noMr3fVPPE)
  * [AdapterHandlerEntry](#noObYdY1fG)
  * [AdapterFingerPrint](#nojJqrl3M4)
  * [AdapterHandlerTable](#noxkkAx-Fs)
  * [AdapterHandlerTableIterator](#no_uzGgg03)
  * [MethodArityHistogram](#nor3AfbgKY)


---
## <a name="noQwiO5QYM" id="noQwiO5QYM">SharedRuntime</a>

### 概要(Summary)
HotSpot 内にある Runtime クラスの1つ 
(= アセンブリでは書くことが難しい複雑な機能(例外ハンドリング, 重量ロック処理, etc)を納めた名前空間(AllStatic クラス))
(See: [here](no1904gX2.html) for details).

その中でも SharedRuntime クラスには, 
インタープリタからも JIT 生成コード (C1, C2, Shark) からも共通で利用されるルーチンが納められている.

(ところでコメント中に出ている CompilerRuntime とは?? #TODO)


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.hpp))
    // Runtime is the base class for various runtime interfaces
    // (InterpreterRuntime, CompilerRuntime, etc.). It provides
    // shared functionality such as exception forwarding (C++ to
    // Java exceptions), locking/unlocking mechanisms, statistical
    // information, etc.
    
    class SharedRuntime: AllStatic {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 備考(Notes)
数学関係のメソッドは sharedRuntimeTrans.cpp と sharedRuntimeTrig.cpp で定義されている.

なお, SharedRuntime のメソッド用の補助関数として
"__ieee754_" や "__kernel_" から始まる関数が定義されているが, これらは fdlibm からコピーしてきたもの
(つまり, jdk の java.lang.StrictMath で使用している fdlibm ライブラリ内のコードと同一).

コメントによると, 
「generate_math_entry() や JIT コンパイラ等で数学関数を最適化しようとすると, 
これらの関数はほとんどのプラットフォームで必要になる (x86 は sin/cos の精度が合わないし sparc には sin/cos がない).
その際には libjava.so にジャンプするよりも SharedRuntime 内にコードがあった方が高速なのでコピーした」, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp))
    // This file contains copies of the fdlibm routines used by
    // StrictMath. It turns out that it is almost always required to use
    // these runtime routines; the Intel CPU doesn't meet the Java
    // specification for sin/cos outside a certain limited argument range,
    // and the SPARC CPU doesn't appear to have sin/cos instructions. It
    // also turns out that avoiding the indirect call through function
    // pointer out to libjava.so in SharedRuntime speeds these routines up
    // by roughly 15% on both Win32/x86 and Solaris/SPARC.
```

(以下は sharedRuntimeTrans.cpp 内で定義されている補助関数. これらは log, log10, exp, pow 用)

* __ieee754_log


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp))
    static double __ieee754_log(double x) {
```

* __ieee754_log10


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp))
    static double __ieee754_log10(double x) {
```

* __ieee754_exp


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp))
    static double __ieee754_exp(double x) {
```

* __ieee754_pow


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrans.cpp))
    double __ieee754_pow(double x, double y) {
```

(以下は sharedRuntimeTrig.cpp 内で定義されている補助関数. これらは sin, cos, tan 用)

* __ieee754_rem_pio2


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrig.cpp))
    static SAFEBUF int __ieee754_rem_pio2(double x, double *y) {
```

* __kernel_sin

```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrig.cpp))
    static double __kernel_sin(double x, double y, int iy)
```

* __kernel_cos

```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrig.cpp))
    static double __kernel_cos(double x, double y)
```

* __kernel_tan

```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntimeTrig.cpp))
    static double __kernel_tan(double x, double y, int iy)
```




### 詳細(Details)
See: [here](../doxygen/classSharedRuntime.html) for details

---
## <a name="noMr3fVPPE" id="noMr3fVPPE">AdapterHandlerLibrary</a>

### 概要(Summary)
JIT Compiler 用のクラス. 

以下の 2種類のスタブコードを生成するクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).
(See: [here](no7882a7C.html) and [here](no1904s1R.html) for details).

* インタープリタ実行のメソッドと JIT コードが生成されたメソッドが相互に呼び出し会うためのスタブコード
  ("i2c アダプタ", "c2i アダプタ").

  (インタープリタ実行時と JIT 生成コードでは calling convention が異なるが, 
   これらのスタブコードを介して呼び出すことでその違いを吸収している)

* JIT コンパイルされたネイティブメソッドのメソッドエントリ部として働くスタブコード ("native wrapper").
  
  (というか, ネイティブメソッドの場合は JIT コンパイルされてもメソッドの中身は変わらないので, 
   実質的な違いはこのエントリ部と i2c/c2i アダプタ部しかないわけだが...)
  
  (なおコメントによると, native wrapper が行う処理は adapter にかなり近いのでこのクラスで扱っている, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.hpp))
    // ---------------------------------------------------------------------------
    // Implementation of AdapterHandlerLibrary
    //
    // This library manages argument marshaling adapters and native wrappers.
    // There are 2 flavors of adapters: I2C and C2I.
    //
    // The I2C flavor takes a stock interpreted call setup, marshals the
    // arguments for a Java-compiled call, and jumps to Rmethod-> code()->
    // code_begin().  It is broken to call it without an nmethod assigned.
    // The usual behavior is to lift any register arguments up out of the
    // stack and possibly re-pack the extra arguments to be contigious.
    // I2C adapters will save what the interpreter's stack pointer will be
    // after arguments are popped, then adjust the interpreter's frame
    // size to force alignment and possibly to repack the arguments.
    // After re-packing, it jumps to the compiled code start.  There are
    // no safepoints in this adapter code and a GC cannot happen while
    // marshaling is in progress.
    //
    // The C2I flavor takes a stock compiled call setup plus the target method in
    // Rmethod, marshals the arguments for an interpreted call and jumps to
    // Rmethod->_i2i_entry.  On entry, the interpreted frame has not yet been
    // setup.  Compiled frames are fixed-size and the args are likely not in the
    // right place.  Hence all the args will likely be copied into the
    // interpreter's frame, forcing that frame to grow.  The compiled frame's
    // outgoing stack args will be dead after the copy.
    //
    // Native wrappers, like adapters, marshal arguments.  Unlike adapters they
    // also perform an offical frame push & pop.  They have a call to the native
    // routine in their middles and end in a return (instead of ending in a jump).
    // The native wrappers are stored in real nmethods instead of the BufferBlobs
    // used by the adapters.  The code generation happens here because it's very
    // similar to what the adapters have to do.
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.hpp))
    class AdapterHandlerLibrary: public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* 各 methodOop に "i2c アダプタ", "c2i アダプタ" をセットする処理
  
  methodOopDesc::make_adapters()
  -> AdapterHandlerLibrary::get_adapter()
     -> SharedRuntime::generate_i2c2i_adapters()  (sparc の場合) (x86 の場合) (zero の場合)
        -> AdapterHandlerLibrary::new_entry()

* JIT コンパイルされた各ネイティブメソッドに "native wrapper" をセットする処理
  
  CompileBroker::compile_method()
  -> AdapterHandlerLibrary::create_native_wrapper()

* (#TODO)
  
  DTraceJSDT::activate()
  -> AdapterHandlerLibrary::create_dtrace_nmethod()

* トラブルシューティング用の処理

  os::print_location()
  -> AdapterHandlerLibrary::contains()
  -> AdapterHandlerLibrary::print_handler_on()

* デバッグ用(開発時用)の処理
  
  SharedRuntime::print_statistics()
  -> AdapterHandlerLibrary::print_statistics()
```




### 詳細(Details)
See: [here](../doxygen/classAdapterHandlerLibrary.html) for details

---
## <a name="noObYdY1fG" id="noObYdY1fG">AdapterHandlerEntry</a>

### 概要(Summary)
AdapterHandlerLibrary クラス用の補助クラス.

AdapterHandlerLibrary が生成した "i2c/c2i アダプタ" を管理するためのクラス.
1つの AdapterHandlerEntry オブジェクトが "i2c アダプタ" と "c2i アダプタ" のペア1組に対応する
(あるメソッドに対応する "i2c アダプタ" と "c2i アダプタ" は同時に作られるため, これらはペアで管理される).


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.hpp))
    class AdapterHandlerEntry : public BasicHashtableEntry {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている. 

* AdapterHandlerLibrary クラスの _abstract_method_handler フィールド (static フィールド)
  
  abstract メソッド用の "i2c/c2i アダプタ" を管理する AdapterHandlerEntry
  (abstract メソッドなので使われない気もするが念のために生成しておく, とのこと (See: AdapterHandlerLibrary::initialize()).

* 各 methodOopDesc オブジェクトの _adapter フィールド
  
  その methodOopDesc 用の "i2c/c2i アダプタ" を管理する AdapterHandlerEntry

* AdapterHandlerLibrary クラスの _adapters フィールド (static フィールド)
  
  (正確には, このフィールドは AdapterHandlerTable オブジェクトを格納するフィールド.
  この中に, 生成した全ての AdapterHandlerEntry オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
AdapterHandlerTable::new_entry() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
* 各 methodOop に "i2c アダプタ", "c2i アダプタ" をセットする処理

  methodOopDesc::make_adapters()
  -> AdapterHandlerLibrary::get_adapter()
     -> AdapterHandlerLibrary::initialize()
        -> AdapterHandlerLibrary::new_entry()
           -> AdapterHandlerTable::new_entry()
     -> SharedRuntime::generate_i2c2i_adapters()  (sparc の場合) (x86 の場合) (zero の場合)
        -> AdapterHandlerLibrary::new_entry()
           -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classAdapterHandlerEntry.html) for details

---
## <a name="nojJqrl3M4" id="nojJqrl3M4">AdapterFingerPrint</a>

### 概要(Summary)
AdapterHandlerLibrary クラス内で使用される補助クラス.

引数や返値の情報(個数,型)に基づいたハッシュ値を計算するクラス.
calling convention が同じメソッドについては同じ i2c/c2i スタブが流用できるため, 
それを探すためにこのハッシュ値が用いられている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    // A simple wrapper class around the calling convention information
    // that allows sharing of adapters for the same calling convention.
    class AdapterFingerPrint : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 AdapterHandlerEntry オブジェクトの _fingerprint フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ただし, AdapterHandlerTable::lookup() 内のものは局所変数としての生成なので一時的なオブジェクト).

* AdapterHandlerLibrary::initialize()
* AdapterHandlerLibrary::get_adapter()
* AdapterHandlerTable::lookup() (局所変数として生成)




### 詳細(Details)
See: [here](../doxygen/classAdapterFingerPrint.html) for details

---
## <a name="noxkkAx-Fs" id="noxkkAx-Fs">AdapterHandlerTable</a>

### 概要(Summary)
AdapterHandlerLibrary クラス内で使用される補助クラス.

一度作成した AdapterHandlerEntry をメモイズしておくためのハッシュテーブル.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    // A hashtable mapping from AdapterFingerPrints to AdapterHandlerEntries
    class AdapterHandlerTable : public BasicHashtable {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
AdapterHandlerLibrary クラスの _adapters フィールド (static フィールド) に(のみ)格納されている.

(ただし, AdapterHandlerTable オブジェクトの生成自体は実際に必要になるまで遅延されている)

#### 生成箇所(where its instances are created)
AdapterHandlerLibrary::initialize() 内で(のみ)生成されている (= 初めて使用される時まで生成が遅延されている).
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* 各 methodOop に "i2c アダプタ", "c2i アダプタ" をセットする処理

  methodOopDesc::make_adapters()
  -> AdapterHandlerLibrary::get_adapter()
     -> AdapterHandlerLibrary::initialize()
```




### 詳細(Details)
See: [here](../doxygen/classAdapterHandlerTable.html) for details

---
## <a name="no_uzGgg03" id="no_uzGgg03">AdapterHandlerTableIterator</a>

### 概要(Summary)
AdapterHandlerLibrary クラス内で使用される補助クラス.

AdapterHandlerTable 内の要素をたどるためのイテレータクラス(StackObjクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    class AdapterHandlerTableIterator : public StackObj {
```

### 使われ方(Usage)
#### 使用例(usage examples)
実際に使用する際にはこんな感じになる.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
      AdapterHandlerTableIterator iter(_adapters);
      while (iter.has_next()) {
        AdapterHandlerEntry* a = iter.next();
    ...
      }
```

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* AdapterHandlerLibrary::contains()
* AdapterHandlerLibrary::print_handler_on()




### 詳細(Details)
See: [here](../doxygen/classAdapterHandlerTableIterator.html) for details

---
## <a name="nor3AfbgKY" id="nor3AfbgKY">MethodArityHistogram</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

SharedRuntime クラス内で使用される補助クラス.

これまでに実行されたメソッド呼び出しについて, 
「引数の個数がいくらのメソッドが何回呼び出されたか」という統計情報を表示する.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    #ifndef PRODUCT
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
    class MethodArityHistogram {
```

### 使われ方(Usage)
SharedRuntime::print_call_statistics() 内で(のみ)使用されている.

### 内部構造(Internal structure)
行う処理は以下の通り.


```cpp
    ((cite: hotspot/src/share/vm/runtime/sharedRuntime.cpp))
      MethodArityHistogram() {
        MutexLockerEx mu(CodeCache_lock, Mutex::_no_safepoint_check_flag);
        _max_arity = _max_size = 0;
        for (int i = 0; i < MAX_ARITY; i++) _arity_histogram[i] = _size_histogram [i] = 0;
        CodeCache::nmethods_do(add_method_to_histogram);
        print_histogram();
      }
```




### 詳細(Details)
See: [here](../doxygen/classMethodArityHistogram.html) for details

---
