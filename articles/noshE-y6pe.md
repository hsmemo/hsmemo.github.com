---
layout: default
title: InlineCacheBuffer クラス関連のクラス (ICStub, InlineCacheBuffer)
---
[Top](../index.html)

#### InlineCacheBuffer クラス関連のクラス (ICStub, InlineCacheBuffer)

これらは, CompiledIC 用の補助クラス
(そのため, JIT 生成コード中の invokevirtual/invokeinterface の箇所でしか使われない)
(See: CompiledIC).


```
    ((cite: hotspot/src/share/vm/code/icBuffer.hpp))
    // For CompiledIC's:
    //
    // In cases where we do not have MT-safe state transformation,
    // we go to a transition state, using ICStubs. At a safepoint,
    // the inline caches are transferred from the transitional code:
    //
    //    instruction_address --> 01 set xxx_oop, Ginline_cache_klass
    //                            23 jump_to Gtemp, yyyy
    //                            4  nop
```


### クラス一覧(class list)

  * [InlineCacheBuffer](#no4eb_BiI3)
  * [ICStub](#no_jGC7HvS)


---
## <a name="no4eb_BiI3" id="no4eb_BiI3">InlineCacheBuffer</a>

### 概要(Summary)
(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/icBuffer.hpp))
    class InlineCacheBuffer: public AllStatic {
```

### 内部構造(Internal structure)
(#Under Construction)

InlineCacheBuffer の中には invokevirtual や invokeinterface での dynamic dispatch 処理に使う stub が入っている.

なお, InlineCacheBuffer::ic_stub_code_size() は, 生成する stub の大きさを返す関数 (コード格納用のバッファの確保時に使用).


```
    ((cite: hotspot/src/share/vm/code/icBuffer.hpp))
      static int ic_stub_code_size();
```



### 詳細(Details)
See: [here](../doxygen/classInlineCacheBuffer.html) for details

---
## <a name="no_jGC7HvS" id="no_jGC7HvS">ICStub</a>

### 概要(Summary)
(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/icBuffer.hpp))
    class ICStub: public Stub {
```




### 詳細(Details)
See: [here](../doxygen/classICStub.html) for details

---
