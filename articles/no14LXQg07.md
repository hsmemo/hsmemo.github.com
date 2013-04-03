---
layout: default
title: nmethod クラス関連のクラス (ExceptionCache, PcDescCache, nmethod, nmethodLocker, 及びそれらの補助クラス(DetectScavengeRoot, VerifyOopsClosure, DebugScavengeRoot))
---
[Top](../index.html)

#### nmethod クラス関連のクラス (ExceptionCache, PcDescCache, nmethod, nmethodLocker, 及びそれらの補助クラス(DetectScavengeRoot, VerifyOopsClosure, DebugScavengeRoot))

これらは, JIT コンパイルされた Java メソッドを管理するためのクラス (なお OSR の場合はメソッドの一部分だけのこともある). (See: [here](no7882MiN.html) for details)

("nmethod" とは "native method" の意味. [Glossary] 参照)


### クラス一覧(class list)

  * [nmethod](#noEi72W2DF)
  * [nmethodLocker](#nozr47LyoY)
  * [ExceptionCache](#no9N9NaM_-)
  * [PcDescCache](#no7lb2MswD)
  * [DetectScavengeRoot](#no-c3bq7aM)
  * [VerifyOopsClosure](#no2ipkwCWM)
  * [DebugScavengeRoot](#noYoz1M_MH)


---
## <a name="noEi72W2DF" id="noEi72W2DF">nmethod</a>

### 概要(Summary)
JIT コンパイルされた Java メソッドを表す CodeBlob クラス.


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
    class nmethod : public CodeBlob {
```

### 内部構造(Internal structure)
内部には以下のような情報を含んでいる.

* data array と書かれた領域には, ScopeDesc の元となるデータが詰まった配列がある模様. (See: ScopeDesc)
* pcs と書かれた領域には, PcDesc が詰まった配列がある模様. (See: PcDesc)


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
    // nmethods (native methods) are the compiled code versions of Java methods.
    //
    // An nmethod contains:
    //  - header                 (the nmethod structure)
    //  [Relocation]
    //  - relocation information
    //  - constant part          (doubles, longs and floats used in nmethod)
    //  - oop table
    //  [Code]
    //  - code body
    //  - exception handler
    //  - stub code
    //  [Debugging information]
    //  - oop array
    //  - data array
    //  - pcs
    //  [Exception handler table]
    //  - handler entry point array
    //  [Implicit Null Pointer exception table]
    //  - implicit null table array
```




### 詳細(Details)
See: [here](../doxygen/classnmethod.html) for details

---
## <a name="nozr47LyoY" id="nozr47LyoY">nmethodLocker</a>

### 概要(Summary)
nmethod クラス用のユーティリティ・クラス.

ソースコード中のあるスコープの間だけ, 
指定した nmethod を remove や made not entrant (made zombie) から保護するための一時オブジェクト(StackObjクラス).


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
    // Locks an nmethod so its code will not get removed and it will not
    // be made into a zombie, even if it is a not_entrant method. After the
    // nmethod becomes a zombie, if CompiledMethodUnload event processing
    // needs to be done, then lock_nmethod() is used directly to keep the
    // generated code from being reused too early.
    class nmethodLocker : public StackObj {
```

### 内部構造(Internal structure)
実際にやっているのは nmethod の _lock_count フィールドの増減.

(コンストラクタ内で _lock_count を増加させ, デストラクタ内で減少させている)


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
      nmethodLocker(nmethod *nm) { _nm = nm; lock_nmethod(_nm); }
    ...
      ~nmethodLocker() { unlock_nmethod(_nm); }
```


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    // Only JvmtiDeferredEvent::compiled_method_unload_event()
    // should pass zombie_ok == true.
    void nmethodLocker::lock_nmethod(nmethod* nm, bool zombie_ok) {
      if (nm == NULL)  return;
      Atomic::inc(&nm->_lock_count);
      guarantee(zombie_ok || !nm->is_zombie(), "cannot lock a zombie method");
    }
    
    void nmethodLocker::unlock_nmethod(nmethod* nm) {
      if (nm == NULL)  return;
      Atomic::dec(&nm->_lock_count);
      guarantee(nm->_lock_count >= 0, "unmatched nmethod lock/unlock");
    }
```




### 詳細(Details)
See: [here](../doxygen/classnmethodLocker.html) for details

---
## <a name="no9N9NaM_-" id="no9N9NaM_-">ExceptionCache</a>

### 概要(Summary)
nmethod クラス内で使用される補助クラス.

exception handler の情報をキャッシュしておき,
同じ箇所で同じ例外が発生した際に例外ハンドラの lookup 処理を高速化する.


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
    // This class is used internally by nmethods, to cache
    // exception/pc/handler information.
    
    class ExceptionCache : public CHeapObj {
```

### 使われ方(Usage)
nmethod::handler_for_exception_and_pc() 内で使用されている.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    // public method for accessing the exception cache
    // These are the public access methods.
    address nmethod::handler_for_exception_and_pc(Handle exception, address pc) {
    ...
      ExceptionCache* ec = exception_cache();
      while (ec != NULL) {
        address ret_val;
        if ((ret_val = ec->match(exception,pc)) != NULL) {
          return ret_val;
        }
        ec = ec->next();
      }
      return NULL;
```




### 詳細(Details)
See: [here](../doxygen/classExceptionCache.html) for details

---
## <a name="no7lb2MswD" id="no7lb2MswD">PcDescCache</a>

### 概要(Summary)
nmethod クラス内で使用される補助クラス.

以前に取得した PcDesc の情報をキャッシュしておき,
同じ PcDesc が必要になった際の lookup 処理を高速化する.


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
    // cache pc descs found in earlier inquiries
    class PcDescCache VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
nmethod::find_pc_desc() や nmethod::find_pc_desc_internal() 内で使われている.


```
    ((cite: hotspot/src/share/vm/code/nmethod.hpp))
      PcDesc* find_pc_desc(address pc, bool approximate) {
        PcDesc* desc = _pc_desc_cache.last_pc_desc();
```


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    PcDesc* nmethod::find_pc_desc_internal(address pc, bool approximate) {
    ...
      PcDesc* res = _pc_desc_cache.find_pc_desc(pc_offset, approximate);
    ...
      PcDesc* mid = _pc_desc_cache.last_pc_desc();
```




### 詳細(Details)
See: [here](../doxygen/classPcDescCache.html) for details

---
## <a name="no-c3bq7aM" id="no-c3bq7aM">DetectScavengeRoot</a>

### 概要(Summary)
nmethod::detect_scavenge_root_oops() 内で使用されている補助クラス.

対象の nmethod オブジェクトが保持しているポインタの中に, 
Minor GC の root となるもの (= New 領域を指しているもの) があるかどうかを判定する Closure.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    class DetectScavengeRoot: public OopClosure {
```

### 使われ方(Usage)
Minor GC の root となるものが存在すれば, 
DetectScavengeRoot::detected_scavenge_root() メソッドが true を返す.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    bool nmethod::detect_scavenge_root_oops() {
      DetectScavengeRoot detect_scavenge_root;
      NOT_PRODUCT(if (TraceScavenge)  detect_scavenge_root._print_nm = this);
      oops_do(&detect_scavenge_root);
      return detect_scavenge_root.detected_scavenge_root();
```




### 詳細(Details)
See: [here](../doxygen/classDetectScavengeRoot.html) for details

---
## <a name="no2ipkwCWM" id="no2ipkwCWM">VerifyOopsClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

nmethod::verify() 内で使用される補助クラス.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    // Verification
    
    class VerifyOopsClosure: public OopClosure {
```

### 使われ方(Usage)

```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    void nmethod::verify() {
    ...
      VerifyOopsClosure voc(this);
      oops_do(&voc);
      assert(voc.ok(), "embedded oops must be OK");
```




### 詳細(Details)
See: [here](../doxygen/classVerifyOopsClosure.html) for details

---
## <a name="noYoz1M_MH" id="noYoz1M_MH">DebugScavengeRoot</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

nmethod::verify_scavenge_root_oops() 内で使用される補助クラス.


```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    #ifndef PRODUCT
    
    class DebugScavengeRoot: public OopClosure {
```

### 使われ方(Usage)

```
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    void nmethod::verify_scavenge_root_oops() {
    ...
        // Actually look inside, to verify the claim that it's clean.
        DebugScavengeRoot debug_scavenge_root(this);
        oops_do(&debug_scavenge_root);
        if (!debug_scavenge_root.ok())
          fatal("found an unadvertised bad scavengable oop in the code cache");
```




### 詳細(Details)
See: [here](../doxygen/classDebugScavengeRoot.html) for details

---
