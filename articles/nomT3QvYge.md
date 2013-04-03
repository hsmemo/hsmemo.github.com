---
layout: default
title: compiledICHolderOopDesc クラス 
---
[Top](../index.html)

#### compiledICHolderOopDesc クラス 



---
## <a name="nogthzqMJX" id="nogthzqMJX">compiledICHolderOopDesc</a>

### 概要(Summary)
JIT Compiler 用の補助クラス.
Inline Caching 機能において飛び先が interpreter 実行の場合に使用される
(See: [here](no7882oxz.html) for details).

(飛び先の c2i_unverified_entry_point に渡す引数(token)として使用されている)


```
    ((cite: hotspot/src/share/vm/oops/compiledICHolderOop.hpp))
    // A compiledICHolderOop is a helper object for the inline cache implementation.
    // It holds an intermediate value (method+klass pair) used when converting from
    // compiled to an interpreted call.
    //
    // compiledICHolderOops are always allocated permanent (to avoid traversing the
    // codeCache during scavenge).
    
    
    class compiledICHolderOopDesc : public oopDesc {
```

### 内部構造(Internal structure)
内部には, 以下の2つの情報を格納している.

  * klassOop  _holder_klass
    
    「想定している class」を示す.

    c2i_unverified_entry_point の type check コードが使用する.

  * methodOop _holder_method
    
    type check が通った後で実際にコードを実行するために使用する.


```
    ((cite: hotspot/src/share/vm/oops/compiledICHolderOop.hpp))
      methodOop _holder_method;
      klassOop  _holder_klass;    // to avoid name conflict with oopDesc::_klass
```

### 備考(Notes)
なお, 実際の使用箇所では compiledICHolderOop という別名(もしくはラッパークラス)で使われることが多い (See: compiledICHolderOop).




### 詳細(Details)
See: [here](../doxygen/classcompiledICHolderOopDesc.html) for details

---
