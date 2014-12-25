---
layout: default
title: G1AllocRegion クラス及びその補助クラス (G1AllocRegion, ar_ext_msg)
---
[Top](../index.html)

#### G1AllocRegion クラス及びその補助クラス (G1AllocRegion, ar_ext_msg)

これらは, G1CollectedHeap 使用時のメモリ確保処理を補佐するクラス.


### クラス一覧(class list)

  * [G1AllocRegion](#noRwdNgxny)
  * [ar_ext_msg](#noO7DG73yh)


---
## <a name="noRwdNgxny" id="noRwdNgxny">G1AllocRegion</a>

### 概要(Summary)
G1CollectedHeap 使用時のメモリ確保処理で使用される補助クラス.

オブジェクト確保に使用している HeapRegion を管理するクラスの基底クラス.
メモリ確保時には, この HeapRegion から TLAB が確保される
(See: [here](no289164hI.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

なお, 
確保処理の fast-path は lock free だと想定しており,
lock が必要になるのは region が一杯になったので新しい region に入れ替えるときだけ, 
とのこと.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.hpp))
    // A class that holds a region that is active in satisfying allocation
    // requests, potentially issued in parallel. When the active region is
    // full it will be retired it replaced with a new one. The
    // implementation assumes that fast-path allocations will be lock-free
    // and a lock will need to be taken when the active region needs to be
    // replaced.
    
    class G1AllocRegion VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
確保に使用する HeapRegion は _alloc_region フィールドに格納されている.

(なおコメントによると,
 G1AllocRegion::_alloc_region フィールドは, 
 初期化後 (init()が呼ばれた後) は決して NULL になることはない, 
 とのこと.)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.hpp))
      // The active allocating region we are currently allocating out
      // of. The invariant is that if this object is initialized (i.e.,
      // init() has been called and release() has not) then _alloc_region
      // is either an active allocating region or the dummy region (i.e.,
      // it can never be NULL) and this object can be used to satisfy
      // allocation requests. If this object is not initialized
      // (i.e. init() has not been called or release() has been called)
      // then _alloc_region is NULL and this object should not be used to
      // satisfy allocation requests (it was done this way to force the
      // correct use of init() and release()).
      HeapRegion* _alloc_region;
```




### 詳細(Details)
See: [here](../doxygen/classG1AllocRegion.html) for details

---
## <a name="noO7DG73yh" id="noO7DG73yh">ar_ext_msg</a>

### 概要(Summary)
G1AllocRegion クラス内で使用される補助クラス.

G1AllocRegion 用の FormatBuffer クラス
(err_msg は FormatBuffer の別名 (See: FormatBuffer)).

G1AllocRegion の friend class になっているので, 
G1AllocRegion 内のフィールドの値にもアクセスでき, より詳細なエラー情報が出せる模様.
(See: G1AllocRegion::fill_in_ext_msg()).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/g1/g1AllocRegion.hpp))
    class ar_ext_msg : public err_msg {
```

### 使われ方(Usage)
G1AllocRegion 中の様々な箇所で (主に assert 用の文字列を構築する用途で) 使用されている.




### 詳細(Details)
See: [here](../doxygen/classar__ext__msg.html) for details

---
