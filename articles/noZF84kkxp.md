---
layout: default
title: ciBytecodeStream, ciSignatureStream 及び ciExceptionHandlerStream クラス (ciBytecodeStream, ciSignatureStream, ciExceptionHandlerStream)
---
[Top](../index.html)

#### ciBytecodeStream, ciSignatureStream 及び ciExceptionHandlerStream クラス (ciBytecodeStream, ciSignatureStream, ciExceptionHandlerStream)

これらは, JIT Compiler 用のクラス.
より具体的に言うと, メソッド内の情報 
(メソッド中のバイトコード, メソッドの型を表す文字列(Signature String), メソッドの例外ハンドラ) に対して
iterate 処理するためのユーティリティ・クラス
(See: [here](no7882MiN.html) for details).


### クラス一覧(class list)

  * [ciBytecodeStream](#noBNWNqpDX)
  * [ciSignatureStream](#noncjNCkGS)
  * [ciExceptionHandlerStream](#noCoYq7S2w)


---
## <a name="noBNWNqpDX" id="noBNWNqpDX">ciBytecodeStream</a>

### 概要(Summary)
JIT Compiler の作業中に使われる一時オブジェクト(StackObjクラス).

メソッド中のバイトコードに対して iterate 処理するためのイテレータクラス.

なお, このクラスは Constant Pool の中身にアクセスする機能も提供している.
このため, JIT コンパイラは Constant Pool に関するデータ構造については知る必要がない.

また, HotSpot 独自の _fast 系の bytecode (See: [here](no3059AfB.html) and [here](no7882vBO.html) for details) についても, 
ciBytecodeStream の内部で元々の bytecode に戻している 
(そのため, これらについても JIT コンパイラは知る必要はない).


```cpp
    ((cite: hotspot/src/share/vm/ci/ciStreams.hpp))
    // The class is used to iterate over the bytecodes of a method.
    // It hides the details of constant pool structure/access by
    // providing accessors for constant pool items.  It returns only pure
    // Java bytecodes; VM-internal _fast bytecodes are translated back to
    // their original form during iteration.
    class ciBytecodeStream : StackObj {
```




### 詳細(Details)
See: [here](../doxygen/classciBytecodeStream.html) for details

---
## <a name="noncjNCkGS" id="noncjNCkGS">ciSignatureStream</a>

### 概要(Summary)
JIT Compiler の作業中に使われる一時オブジェクト(StackObjクラス).

メソッドの型を表す文字列(Signature String)に対して iterate 処理するためのイテレータクラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciStreams.hpp))
    // ciSignatureStream
    //
    // The class is used to iterate over the elements of a method signature.
    class ciSignatureStream : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ciTypeFlow::get_start_state()
* ciTypeFlow::StateVector::do_invoke()




### 詳細(Details)
See: [here](../doxygen/classciSignatureStream.html) for details

---
## <a name="noCoYq7S2w" id="noCoYq7S2w">ciExceptionHandlerStream</a>

### 概要(Summary)
JIT Compiler の作業中に使われる一時オブジェクト(StackObjクラス).

メソッドの例外ハンドラ(Exception Handler)情報に対して iterate 処理するためのイテレータクラス.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciStreams.hpp))
    // ciExceptionHandlerStream
    //
    // The class is used to iterate over the exception handlers of
    // a method.
    class ciExceptionHandlerStream : public StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ciMethodBlocks::ciMethodBlocks()
* ciTypeFlow::Block::compute_exceptions()
* MethodLiveness::init_basic_blocks()
* XHandlers::XHandlers()  (C1 の場合)
* Parse::catch_inline_exceptions()  (C2 の場合)
* SharkTopLevelBlock::compute_exceptions()  (Shark の場合)




### 詳細(Details)
See: [here](../doxygen/classciExceptionHandlerStream.html) for details

---
