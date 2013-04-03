---
layout: default
title: OopClosure の様々なサブクラス (OopsInGenClosure, ScanClosure, FastScanClosure, FilteringClosure, ScanWeakRefClosure, VerifyOopClosure)
---
[Top](../index.html)

#### OopClosure の様々なサブクラス (OopsInGenClosure, ScanClosure, FastScanClosure, FilteringClosure, ScanWeakRefClosure, VerifyOopClosure)

これらは, 主に GenCollectedHeap 内を処理するための Closure クラス.

なお, これらの内の半分は OopsInGenClosure とそのサブクラス
(OopsInGenClosure のサブクラスは gc_implementation/ 以下にも存在している).



### クラス一覧(class list)

  * [OopsInGenClosure](#noU3Wbuj2F)
  * [ScanClosure](#noVU-XZV1x)
  * [FastScanClosure](#noLZFGj7q-)
  * [FilteringClosure](#nopmd1RG5Y)
  * [ScanWeakRefClosure](#noz406Kzg8)
  * [VerifyOopClosure](#no-Wq1Yi4H)


---
## <a name="noU3Wbuj2F" id="noU3Wbuj2F">OopsInGenClosure</a>

### 概要(Summary)
GenCollectedHeap の処理で使用される補助クラス. (See: [here](no28916sKh.html) for details)

指定された Generation 内の oop をスキャンする Closure クラス (の基底クラス).

なお, このクラスのサブクラスを作る際には, do_oop() メソッドの最後で
OopsInGenClosure::do_barrier() を呼び出すようにしないといけない, とのこと.

また, do_oop() がオーバーライドできていないためこのクラス自体は abstract class, とのこと.

```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    // Closure for iterating roots from a particular generation
    // Note: all classes deriving from this MUST call this do_barrier
    // method at the end of their own do_oop method!
    // Note: no do_oop defined, this is an abstract class.
    
    class OopsInGenClosure : public OopClosure {
```




### 詳細(Details)
See: [here](../doxygen/classOopsInGenClosure.html) for details

---
## <a name="noVU-XZV1x" id="noVU-XZV1x">ScanClosure</a>

### 概要(Summary)
GenCollectedHeap の処理で使用される補助クラス.

GenCollectedHeap に対する Minor GC 処理で使用される Closure クラス.

まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    // Closure for scanning DefNewGeneration.
    //
    // This closure will perform barrier store calls for ALL
    // pointers in scanned oops.
    class ScanClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
ParNewGeneration::collect() の中で使われている.

(その他に EvacuateFollowersClosure オブジェクト内でも使われているようだが,
 そもそも EvacuateFollowersClosure クラス自体が...
 (See: EvacuateFollowersClosure))

### 内部構造(Internal structure)
処理自体は FastScanClosure とよく似ている.

ただし, このクラスの場合は処理対象となった全ての oop に対して OopsInGenClosure::do_barrier() を呼び出す.
(See: FastScanClosure)




### 詳細(Details)
See: [here](../doxygen/classScanClosure.html) for details

---
## <a name="noLZFGj7q-" id="noLZFGj7q-">FastScanClosure</a>

### 概要(Summary)
GenCollectedHeap の処理で使用される補助クラス.

DefNewGeneration 用の Minor GC 処理で使われるクロージャークラス.

まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    // Closure for scanning DefNewGeneration.
    //
    // This closure only performs barrier store calls on
    // pointers into the DefNewGeneration. This is less
    // precise, but faster, than a ScanClosure
    class FastScanClosure: public OopsInGenClosure {
    
```

### 使われ方(Usage)
DefNewGeneration::collect() 内で(のみ)使用されている (See: [here](no2114uZg.html) for details).

### 内部構造(Internal structure)
処理自体は ScanClosure とよく似ている. (See: ScanClosure)

ただし, このクラスの場合は DefNewGeneration 内を指しているポインタでなければ
OopsInGenClosure::do_barrier() を呼び出さない.
このため, 正確さという面では少し劣るが, ScanClosure よりも高速に処理できる.




### 詳細(Details)
See: [here](../doxygen/classFastScanClosure.html) for details

---
## <a name="nopmd1RG5Y" id="nopmd1RG5Y">FilteringClosure</a>

### 概要(Summary)
Filtering_DCTOC クラス(とそのサブクラス)の処理で使用される補助クラス (See: Filtering_DCTOC).

他の OopClosure と組み合わせて使用される Closure クラス.
「指定した OopClosure の効果をあるアドレス以下の範囲のみに限定したい」という場合に使用される.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    class FilteringClosure: public OopClosure {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
Filtering_DCTOC::walk_mem_region() 内で(のみ)生成されている.

(ここで局所変数として生成されている.
このオブジェクトが後述の Filtering_DCTOC::walk_mem_region_with_cl() に渡されて使用されている)

#### 使用箇所(where its instances are used)
Filtering_DCTOC::walk_mem_region_with_cl() が引数として FilteringClosure 型の値を受け取る.
ただし, Filtering_DCTOC::walk_mem_region_with_cl() 自体は abstract method であり,
実際に使用するのはサブクラスのメソッド.

* ContiguousSpaceDCTOC:
  * ContiguousSpaceDCTOC::walk_mem_region_with_cl()
* FreeListSpace_DCTOC (CMS 用):
  * FreeListSpace_DCTOC::walk_mem_region_with_cl()
  * FreeListSpace_DCTOC::walk_mem_region_with_cl_par()
  * FreeListSpace_DCTOC::walk_mem_region_with_cl_nopar()
* HeapRegionDCTOC (G1GC 用):
  * HeapRegionDCTOC::walk_mem_region_with_cl()

### 内部構造(Internal structure)
処理自体は, コンストラクタ引数で渡された OopClosure を呼び出すだけ.

ただし, 処理対象のアドレスが (同じくコンストラクタ引数で渡された) boundary を超えていれば呼び出さない.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
      FilteringClosure(HeapWord* boundary, OopClosure* cl) :
```




### 詳細(Details)
See: [here](../doxygen/classFilteringClosure.html) for details

---
## <a name="noz406Kzg8" id="noz406Kzg8">ScanWeakRefClosure</a>

### 概要(Summary)
GenCollectedHeap の処理で使用される補助クラス.

GenCollectedHeap に対する Minor GC 処理で使用される Closure クラス.
weak reference を処理する為に使われる.

なお, 行う処理の内容は ScanClosure によく似ているが, 
こちらは OopsInGenClosure ではなく OopClosure のサブクラスになっている.

```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    // Closure for scanning DefNewGeneration's weak references.
    // NOTE: very much like ScanClosure but not derived from
    //  OopsInGenClosure -- weak references are processed all
    //  at once, with no notion of which generation they were in.
    class ScanWeakRefClosure: public OopClosure {
```

### 使われ方(Usage)
DefNewGeneration::collect(), 及び ParNewGeneration::collect() 内で(のみ)使用されている.
(See: [here](no2114uZg.html) for details)

### 内部構造(Internal structure)
処理自体は ScanClosure とよく似ている. (See: ScanClosure)

ただし, このクラスの場合は do_oop() の最後に OopsInGenClosure::do_barrier() を呼び出さない.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.inline.hpp))
    // Note similarity to ScanClosure; the difference is that
    // the barrier set is taken care of outside this closure.
```




### 詳細(Details)
See: [here](../doxygen/classScanWeakRefClosure.html) for details

---
## <a name="no-Wq1Yi4H" id="no-Wq1Yi4H">VerifyOopClosure</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス.

指定のメモリ領域中に invalid な oop が無いことを確かめるための Closure.


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
    class VerifyOopClosure: public OopClosure {
```

### 使われ方(Usage)
以下の関数内で使用されている.

* Deoptimization::unpack_frames() : (#ifndef PRODUCT 時, かつ VerifyStack オプションが指定されている場合)
* CodeCache::verify_oops()
* frame::verify()
* JavaThread::verify()
* VMThread::verify()

### 内部構造(Internal structure)
oopDesc::is_oop_or_null() を呼んで invalid かどうかを確認している
(false を返す場合は invalid oop).


```
    ((cite: hotspot/src/share/vm/memory/genOopClosures.hpp))
      template <class T> inline void do_oop_work(T* p) {
        oop obj = oopDesc::load_decode_heap_oop(p);
        guarantee(obj->is_oop_or_null(), err_msg("invalid oop: " INTPTR_FORMAT, (oopDesc*) obj));
      }
```




### 詳細(Details)
See: [here](../doxygen/classVerifyOopClosure.html) for details

---
