---
layout: default
title: ciObject クラス 
---
[Top](../index.html)

#### ciObject クラス 



---
## <a name="no3ZONdXA0" id="no3ZONdXA0">ciObject</a>

### 概要(Summary)
JIT コンパイル処理で使用される一時オブジェクト(ResourceObjクラス) の基底クラス.
JIT Compiler から oop を操作するためのクラス.

JIT コンパイル処理を行う CompilerThread はネイティブ状態(_thread_in_native)で動作するため, 
普通に oop にアクセスしていると GC 処理(safepoint 処理)と競合した場合にダングリングポインタになる恐れがある.
そこで ciObject は, 内部的に JNI 参照 (JNI handle) を用いることで oop を間接参照している.
このため ciObject 経由でアクセスしていれば GC が起こっても安全であることが保証される (See: [here](no7882MiN.html) for details).

なお, ciObject クラスは oopDesc クラスに対応し, そのサブクラスが oop や Klass のサブクラスにほぼ1対1対応する.
ただし, VM 内部では oop と Klass の階層は分かれているが, 
ciObject では klassOop と Klass の区別はない.
代わりに, Klass のサブクラスは直接 ciKlass のサブクラスとして表現している (See: ciKlass).


```
    ((cite: hotspot/src/share/vm/ci/ciObject.hpp))
    // ciObject
    //
    // This class represents an oop in the HotSpot virtual machine.
    // Its subclasses are structured in a hierarchy which mirrors
    // an aggregate of the VM's oop and klass hierarchies (see
    // oopHierarchy.hpp).  Each instance of ciObject holds a handle
    // to a corresponding oop on the VM side and provides routines
    // for accessing the information in its oop.  By using the ciObject
    // hierarchy for accessing oops in the VM, the compiler ensures
    // that it is safe with respect to garbage collection; that is,
    // GC and compilation can proceed independently without
    // interference.
    //
    // Within the VM, the oop and klass hierarchies are separate.
    // The compiler interface does not preserve this separation --
    // the distinction between `klassOop' and `Klass' are not
    // reflected in the interface and instead the Klass hierarchy
    // is directly modeled as the subclasses of ciKlass.
    class ciObject : public ResourceObj {
```

### 内部構造(Internal structure)
定義されているフィールドは以下の通り.

 * jobject  _handle;

   対応する oop を指す JNI 参照 (JNI handle).

 * ciKlass* _klass;
   
   対応する oop のクラスを示す ciKlass オブジェクト.

 * uint     _ident;
   
   この ciObject を差し示す一意な番号. (値は ciObjectFactory::init_ident_of() でセットされる)


```
    ((cite: hotspot/src/share/vm/ci/ciObject.hpp))
      // A JNI handle referring to an oop in the VM.  This
      // handle may, in a small set of cases, correctly be NULL.
      jobject  _handle;
      ciKlass* _klass;
      uint     _ident;
```




### 詳細(Details)
See: [here](../doxygen/classciObject.html) for details

---
