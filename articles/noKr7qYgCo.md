---
layout: default
title: Stub クラス関連のクラス (Stub, StubInterface, StubQueue)
---
[Top](../index.html)

#### Stub クラス関連のクラス (Stub, StubInterface, StubQueue)

これらは, 動的に生成されるマシン語コード片を管理するためのクラス. 特に, 「生成後に滅多に破棄されないコード片」を担当する.


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    // The classes in this file provide a simple framework for the
    // management of little pieces of machine code - or stubs -
    // created on the fly and frequently discarded. In this frame-
    // work stubs are stored in a queue.
```


### クラス一覧(class list)

  * [Stub](#noaYecZA1i)
  * [StubInterface](#noDwRMw-wa)
  * [StubQueue](#nodF4YABVs)


---
## <a name="noaYecZA1i" id="noaYecZA1i">Stub</a>

### 概要(Summary)
動的に生成されるマシンコード片を管理するクラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    class Stub VALUE_OBJ_CLASS_SPEC {
```

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

  * ICStub
  * InterpreterCodelet

### 内部構造(Internal structure)
Stub の中は以下のような構造になっている
(なお, data section や code section は空であってもいい).


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    // Stub serves as abstract base class. A concrete stub
    // implementation is a subclass of Stub, implementing
    // all (non-virtual!) functions required sketched out
    // in the Stub class.
    //
    // A concrete stub layout may look like this (both data
    // and code sections could be empty as well):
    //
    //                ________
    // stub       -->|        | <--+
    //               |  data  |    |
    //               |________|    |
    // code_begin -->|        |    |
    //               |        |    |
    //               |  code  |    | size
    //               |        |    |
    //               |________|    |
    // code_end   -->|        |    |
    //               |  data  |    |
    //               |________|    |
    //                          <--+
```




### 詳細(Details)
See: [here](../doxygen/classStub.html) for details

---
## <a name="noDwRMw-wa" id="noDwRMw-wa">StubInterface</a>

### 概要(Summary)
Stub クラスのサブクラスのメソッドを呼び出すためのラッパークラス.

コメントによると
「Stub 自体のメソッドを virtual にすると Stub に vtable がついてしまってもったいないので, 
それぞれの Stub のサブクラス毎に対応する StubInterface というクラスを作っている」とのこと.

StubInterface のメソッドは virtual となっており, そこから対応する Stub サブクラスのメソッドが呼び出される.


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    // A stub interface defines the interface between a stub queue
    // and the stubs it queues. In order to avoid a vtable and
    // (and thus the extra word) in each stub, a concrete stub
    // interface object is created and associated with a stub
    // buffer which in turn uses the stub interface to interact
    // with its stubs.
    //
    // StubInterface serves as an abstract base class. A concrete
    // stub interface implementation is a subclass of StubInterface,
    // forwarding its virtual function calls to non-virtual calls
    // of the concrete stub (see also macro below). There's exactly
    // one stub interface instance required per stub queue.
    
    class StubInterface: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのは以下のサブクラス.

  * ICStubInterface (ICStub 用)
  * InterpreterCodeletInterface (InterpreterCodelet	用)


### 内部構造(Internal structure)
以下のマクロを使って, それぞれの Stub サブクラスに対応する StubInterface クラスが生成されている.

これで定義される StubInterface サブクラス内では, (Stub クラスのメソッドを virtual にする代わりに)
cast() メソッドでそれぞれのサブクラスにキャストしてから呼び出している.
 
例: `cast(self)->size();`

この cast() はマクロにより型が確定しているので, コンパイル段階でそれぞれの Stub サブクラスのメソッドにリンクされる.


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    // DEF_STUB_INTERFACE is used to create a concrete stub interface
    // class, forwarding stub interface calls to the corresponding
    // stub calls.
    
    #define DEF_STUB_INTERFACE(stub)                           \
      class stub##Interface: public StubInterface {            \
       private:                                                \
        static stub*    cast(Stub* self)                       { return (stub*)self; }                 \
                                                               \
       public:                                                 \
        /* Initialization/finalization */                      \
        virtual void    initialize(Stub* self, int size)       { cast(self)->initialize(size); }       \
        virtual void    finalize(Stub* self)                   { cast(self)->finalize(); }             \
                                                               \
        /* General info */                                     \
        virtual int     size(Stub* self) const                 { return cast(self)->size(); }          \
        virtual int     code_size_to_size(int code_size) const { return stub::code_size_to_size(code_size); } \
                                                               \
        /* Code info */                                        \
        virtual address code_begin(Stub* self) const           { return cast(self)->code_begin(); }    \
        virtual address code_end(Stub* self) const             { return cast(self)->code_end(); }      \
                                                               \
        /* Debugging */                                        \
        virtual void    verify(Stub* self)                     { cast(self)->verify(); }               \
        virtual void    print(Stub* self)                      { cast(self)->print(); }                \
      };
```




### 詳細(Details)
See: [here](../doxygen/classStubInterface.html) for details

---
## <a name="nodF4YABVs" id="nodF4YABVs">StubQueue</a>

### 概要(Summary)
生成した Stub を格納しておくためのキュー.


```cpp
    ((cite: hotspot/src/share/vm/code/stubs.hpp))
    // A StubQueue maintains a queue of stubs.
    // Note: All sizes (spaces) are given in bytes.
    
    class StubQueue: public CHeapObj {
```

### 使われ方(Usage)
各 Stub サブクラスは, それぞれ以下の StubQueue に格納されて管理されている.

  * InlineCacheBuffer::_buffer (ICStub 用)
  * AbstractInterpreter::_code (InterpreterCodelet 用)

### 内部構造(Internal structure)
Stub のメモリは StubQueue 内に確保されたリングバッファ内に前から順に(キュー方式で)確保していく模様.




### 詳細(Details)
See: [here](../doxygen/classStubQueue.html) for details

---
