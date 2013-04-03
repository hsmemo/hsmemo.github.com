---
layout: default
title: BytecodeStream クラス関連のクラス (BaseBytecodeStream, RawBytecodeStream, BytecodeStream)
---
[Top](../index.html)

#### BytecodeStream クラス関連のクラス (BaseBytecodeStream, RawBytecodeStream, BytecodeStream)

これらは, メソッド中の各バイトコード命令の情報にアクセスするためのユーティリティ・クラス.

### 概要(Summary)
これらのクラスは, 指定した範囲の bytecode に対して iterate 処理を行うためのイテレータクラス.

利用する際には, 以下のコメント中の "Usage:" のように使う.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeStream.hpp))
    // A BytecodeStream is used for fast iteration over the bytecodes
    // of a methodOop.
    //
    // Usage:
    //
    // BytecodeStream s(method);
    // Bytecodes::Code c;
    // while ((c = s.next()) >= 0) {
    //   ...
    // }
```

なお, 基底クラスである BaseBytecodeStream には以下の二つのサブクラスがある.
RawBytecodeStream の方が (rewrite の影響を考えない分だけ) 速い模様.

  * RawBytecodeStream : rewritten されていないことが保証されている場合用 (verifier や rewriter 用の簡略化バージョン)
  * BytecodeStream : 一般の場合用


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeStream.hpp))
    // A RawBytecodeStream is a simple version of BytecodeStream.
    // It is used ONLY when we know the bytecodes haven't been rewritten
    // yet, such as in the rewriter or the verifier.
```


### クラス一覧(class list)

  * [BaseBytecodeStream](#nosqND9O1x)
  * [RawBytecodeStream](#noLOgquo0d)
  * [BytecodeStream](#no12-2dz7X)


---
## <a name="nosqND9O1x" id="nosqND9O1x">BaseBytecodeStream</a>

### 概要(Summary)
指定範囲の bytecode をたどるためのイテレータクラス(StackObjクラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeStream.hpp))
    // Here is the common base class for both RawBytecodeStream and BytecodeStream:
    class BaseBytecodeStream: StackObj {
```



### 詳細(Details)
See: [here](../doxygen/classBaseBytecodeStream.html) for details

---
## <a name="noLOgquo0d" id="noLOgquo0d">RawBytecodeStream</a>

### 概要(Summary)
BaseBytecodeStream クラスの具象サブクラスの1つ.
こちらは rewrite 処理について考慮せずにイテレートする.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeStream.hpp))
    class RawBytecodeStream: public BaseBytecodeStream {
```



### 詳細(Details)
See: [here](../doxygen/classRawBytecodeStream.html) for details

---
## <a name="no12-2dz7X" id="no12-2dz7X">BytecodeStream</a>

### 概要(Summary)
BaseBytecodeStream クラスの具象サブクラスの1つ.
こちらは rewrite 処理についても考慮してイテレートする.


```
    ((cite: hotspot/src/share/vm/interpreter/bytecodeStream.hpp))
    // In BytecodeStream, non-java bytecodes will be translated into the
    // corresponding java bytecodes.
    
    class BytecodeStream: public BaseBytecodeStream {
```




### 詳細(Details)
See: [here](../doxygen/classBytecodeStream.html) for details

---
