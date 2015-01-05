---
layout: default
title: ciSymbol クラス 
---
[Top](../index.html)

#### ciSymbol クラス 



---
## <a name="noDOdzzvcN" id="noDOdzzvcN">ciSymbol</a>

### 概要(Summary)
JIT Compiler から Symbol オブジェクトにアクセスするための一時オブジェクト(ResourceObjクラス).
1つの ciSymbol オブジェクトが 1つの Symbol オブジェクトに対応する.

(なお, ciSymbol オブジェクトを生成すると対応する Symbol オブジェクトの reference count がインクリメントされる.
 このため使用中に削除されることはない)


```cpp
    ((cite: hotspot/src/share/vm/ci/ciSymbol.hpp))
    // ciSymbol
    //
    // This class represents a Symbol* in the HotSpot virtual
    // machine.
    class ciSymbol : public ResourceObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に格納されている (#TODO 他の箇所).

* ciObjectFactory クラスの _shared_ci_symbols フィールド (static フィールド)
  
  vmSymbols 内の全ての Symbol オブジェクトに対応する ciSymbol を納めたフィールド.
  
  (正確には, このフィールドは ciSymbol の配列を格納するフィールド.
  この中に, vmSymbols 内の全ての Symbol に対応する ciSymbol オブジェクトが格納されている)
  
  なお, このフィールドは現在は以下のパスで(のみ)参照されている.

<div class="flow-abst"><pre>
  ciEnv::get_symbol()
  -&gt; ciObjectFactory::get_symbol()
     -&gt; ciObjectFactory::vm_symbol_at()
</pre></div>

* 各 ciObjectFactory オブジェクトの _symbols フィールド
  
  生成した ciSymbol を記録しておくためのフィールド
  (なお, 記録しているのはインクリメントした reference count を最後に戻す必要があるため).

  (正確には, このフィールドは ciSymbol の GrowableArray を格納するフィールド.
  この中に, ciObjectFactory::get_symbol() で生成した全ての ciSymbol オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ciObjectFactory::init_shared_objects()
  
  (ciObjectFactory::_shared_ci_symbols フィールドの初期化用)

* ciObjectFactory::get_symbol()
  
  (ファクトリメソッド)

### 内部構造(Internal structure)
コンストラクタ内で, 対応する Symbol オブジェクトの reference count がインクリメントされる.


```cpp
    ((cite: hotspot/src/share/vm/ci/ciSymbol.cpp))
    // ------------------------------------------------------------------
    // ciSymbol::ciSymbol
    //
    // Preallocated symbol variant.  Used with symbols from vmSymbols.
    ciSymbol::ciSymbol(Symbol* s, vmSymbols::SID sid)
      : _symbol(s), _sid(sid)
    {
      assert(_symbol != NULL, "adding null symbol");
      _symbol->increment_refcount();  // increment ref count
      assert(sid_ok(), "must be in vmSymbols");
    }
    
    // Normal case for non-famous symbols.
    ciSymbol::ciSymbol(Symbol* s)
      : _symbol(s), _sid(vmSymbols::NO_SID)
    {
      assert(_symbol != NULL, "adding null symbol");
      _symbol->increment_refcount();  // increment ref count
      assert(sid_ok(), "must not be in vmSymbols");
    }
```

なお, reference count を元に戻す処理は, 現在は以下のパスで(のみ)行われている.

<div class="flow-abst"><pre>
ciEnv::~ciEnv()
-&gt; ciObjectFactory::remove_symbols()
</pre></div>




### 詳細(Details)
See: [here](../doxygen/classciSymbol.html) for details

---
