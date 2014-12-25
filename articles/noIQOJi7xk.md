---
layout: default
title: MarkSweep クラス関連のクラス (MarkSweep, MarkSweep::FollowRootClosure, MarkSweep::MarkAndPushClosure, MarkSweep::FollowStackClosure, MarkSweep::AdjustPointerClosure, MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, PreservedMark, 及びそれらの補助クラス(AdjusterTracker))
---
[Top](../index.html)

#### MarkSweep クラス関連のクラス (MarkSweep, MarkSweep::FollowRootClosure, MarkSweep::MarkAndPushClosure, MarkSweep::FollowStackClosure, MarkSweep::AdjustPointerClosure, MarkSweep::IsAliveClosure, MarkSweep::KeepAliveClosure, PreservedMark, 及びそれらの補助クラス(AdjusterTracker))

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, Mark Sweep Compact 処理を行うクラス (See: [here](no3718kvd.html) for details).


### クラス一覧(class list)

  * [MarkSweep](#noo3Lf6tYS)
  * [MarkSweep::FollowRootClosure](#noUDbAnrB1)
  * [MarkSweep::MarkAndPushClosure](#noaFPG78MH)
  * [MarkSweep::FollowStackClosure](#no_Dx1UFex)
  * [MarkSweep::AdjustPointerClosure](#noD6fBYH6L)
  * [MarkSweep::IsAliveClosure](#nouvPLFy1t)
  * [MarkSweep::KeepAliveClosure](#noEeWuW0jh)
  * [PreservedMark](#noK-RrOy4L)
  * [AdjusterTracker](#noTd_lSKHu)


---
## <a name="noo3Lf6tYS" id="noo3Lf6tYS">MarkSweep</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理で使用される補助関数や Closure クラス等を納めた名前空間(AllStatic クラス).

Mark Sweep Compact 用の関数や Closure 等を納めたクラスは使用する GC アルゴリズムによって異なるが,
このクラスは使用するヒープクラスが GenCollectedHeap の場合に使用される
(とコメントでは書かれているが, 実際には ParallelScavengeHeap や G1CollectedHeap の場合にも若干使用されている).

処理は 4つのフェーズからなる.
なお, クラスのアンロードは Full GC の際にのみ行われる.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
    // MarkSweep takes care of global mark-compact garbage collection for a
    // GenCollectedHeap using a four-phase pointer forwarding algorithm.  All
    // generations are assumed to support marking; those that can also support
    // compaction.
    //
    // Class unloading will only occur when a full gc is invoked.
```


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
    class MarkSweep : AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classMarkSweep.html) for details

---
## <a name="noUDbAnrB1" id="noUDbAnrB1">MarkSweep::FollowRootClosure</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理の phase 1 で使われる Closure クラス.
まだマークが付いていないオブジェクトに対して, 
マークを付け, さらにそこから辿れるものについても再帰的に処理を行う.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      class FollowRootClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラスの MarkSweep::follow_root_closure フィールドに(のみ)格納されている.

(なお, MarkSweep クラスを継承したサブクラスの同名のフィールドにも存在する)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::FollowRootClosure  MarkSweep::follow_root_closure;
```



### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1FollowRootClosure.html) for details

---
## <a name="noaFPG78MH" id="noaFPG78MH">MarkSweep::MarkAndPushClosure</a>

Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理の phase 1 で使われる Closure クラス.
まだマークが付いていないオブジェクトに対して, 
マークを付け, そこから辿れるものを marking stack にプッシュする.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      class MarkAndPushClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラスの MarkSweep::mark_and_push_closure フィールドに(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::MarkAndPushClosure MarkSweep::mark_and_push_closure;
```

#### 使用箇所(where its instances are used)
以下の箇所で使用されている.

* instanceKlassKlass::oop_follow_contents()
* PSMarkSweep::mark_sweep_phase1()




### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1MarkAndPushClosure.html) for details

---
## <a name="no_Dx1UFex" id="no_Dx1UFex">MarkSweep::FollowStackClosure</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理の phase 1 で使われる Closure クラス.
marking stack に溜まっているポインタに対して, 
そこから辿れるもの全てに再帰的に処理を行う.

(現状では, 参照オブジェクト(java.lang.ref オブジェクト)に対する処理にしか用いられていないが...)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      class FollowStackClosure: public VoidClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラスの MarkSweep::follow_stack_closure フィールドに(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::FollowStackClosure MarkSweep::follow_stack_closure;
```

#### 使用箇所(where its instances are used)
現在は, ReferenceProcessor::process_discovered_references() 内で(のみ)使用されている??

(ReferenceProcessor::process_discovered_references() 内では, 見つけたポインタを再帰的に辿っていく処理で使用している模様)




### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1FollowStackClosure.html) for details

---
## <a name="noD6fBYH6L" id="noD6fBYH6L">MarkSweep::AdjustPointerClosure</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理の phase 3 で使われる Closure クラス.
ポインタの値をコンパクション先の新しいアドレスへと書き換える処理を行う.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      class AdjustPointerClosure: public OopsInGenClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラス内の以下の static フィールドに(のみ)格納されている.

  * MarkSweep::adjust_root_pointer_closure フィールド
  * MarkSweep::adjust_pointer_closure フィールド

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::AdjustPointerClosure MarkSweep::adjust_root_pointer_closure(true);
    MarkSweep::AdjustPointerClosure MarkSweep::adjust_pointer_closure(false);
```




### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1AdjustPointerClosure.html) for details

---
## <a name="nouvPLFy1t" id="nouvPLFy1t">MarkSweep::IsAliveClosure</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
IsAliveClosure::do_object_b() メソッドが呼ばれると, 
処理対象のオブジェクトが生きているかどうか(mark されているかどうか)を返す.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      // Used for java/lang/ref handling
      class IsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラスの MarkSweep::is_alive フィールドに(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::IsAliveClosure   MarkSweep::is_alive;
```




### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1IsAliveClosure.html) for details

---
## <a name="noEeWuW0jh" id="noEeWuW0jh">MarkSweep::KeepAliveClosure</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

参照オブジェクト(java.lang.ref オブジェクト)に対する処理に用いられる Closure クラス.
まだマークが付いていないオブジェクトに対して, 
マークを付け, そこから辿れるものを marking stack にプッシュする.

(処理自体は MarkAndPushClosure と同じように見えるが... 何が違う?? #TODO)

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      class KeepAliveClosure: public OopClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MarkSweep クラスの MarkSweep::keep_alive フィールドに(のみ)格納されている.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    MarkSweep::KeepAliveClosure MarkSweep::keep_alive;
```




### 詳細(Details)
See: [here](../doxygen/classMarkSweep_1_1KeepAliveClosure.html) for details

---
## <a name="noK-RrOy4L" id="noK-RrOy4L">PreservedMark</a>

### 概要(Summary)
Mark Sweep Compact 型の Garbage Collection 内で使用される補助クラス.

Mark Sweep Compact 処理中に, 元の mark 値を保存しておくためのクラス.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
    class PreservedMark VALUE_OBJ_CLASS_SPEC {
```

(GC 処理では, 生きているオブジェクトを見つけると, 生きているということを示すために印を付けていく.
この印はオブジェクトの mark フィールド (markOopDesc) に書き込まれるが,
そのままだと元の mark フィールドの値は失われてしまう. 
このため元の mark をどこかに待避しておいて GC 後に書き戻してやる必要がある) (See: markOopDesc)

(なお, mark 値がデフォルトのままのオブジェクトについては待避する必要が無いので (当然ながら) この処理は不要)
(See: markOopDesc::must_be_preserved())

### 使われ方(Usage)
* MarkSweep クラス内には, GC 中に mark 値を待避しておく場所が 2つ用意されている.

  * _preserved_mark_stack および _preserved_oop_stack :
    これらは Stack<*> であり, 二本のスタックを用いて mark と oop のペアを格納していく.
  * _preserved_marks :
    これは PreservedMark の配列.
    配列の最大長は _preserved_count_max に格納される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
      // Space for storing/restoring mark word
      static Stack<markOop>                  _preserved_mark_stack;
      static Stack<oop>                      _preserved_oop_stack;
      static size_t                          _preserved_count;
      static size_t                          _preserved_count_max;
      static PreservedMark*                  _preserved_marks;
```


* _preserved_marks の方は, GC 時に空いている領域を使って領域を確保する.

  _preserved_marks は新しいメモリ確保が必要ないので (後始末等も含めて) 処理は速い.
  しかし長さが足りるかどうかは分からない.
  (See: GenMarkSweep::allocate_stacks(), PSMarkSweep::allocate_stacks(), )

* このため, まずは _preserved_marks の方を使い,
  足りなくなったら _preserved_mark_stack と _preserved_oop_stack を使う.
  (See: MarkSweep::preserve_mark())

* 待避しておいた mark 値については, GC 処理の完了時に呼ばれる
  MarkSweep::restore_marks() 内で, 元のオブジェクトの mark 位置へと書き戻される.

  (なお, その中に含まれるポインタについても, MarkSweep::adjust_marks() で適切に修正されている)
  (See: MarkSweep::adjust_marks(), MarkSweep::restore_marks())




### 詳細(Details)
See: [here](../doxygen/classPreservedMark.html) for details

---
## <a name="noTd_lSKHu" id="noTd_lSKHu">AdjusterTracker</a>

デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない). 
(なお, 正確に言うと「#ifdef VALIDATE_MARK_SWEEP 時にしか定義されない」. ただし, VALIDATE_MARK_SWEEP マクロ定数は #ifdef ASSERT 時にだけ #define されるので同義)

oopDesc::adjust_pointers() でのポインタの修正処理が正しく行われているかどうかを検証する.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    #ifdef VALIDATE_MARK_SWEEP
    ...
    class AdjusterTracker: public OopClosure {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. まず MarkSweep::track_interior_pointers() を呼び出す.

   (この中で, AdjusterTracker によって, 処理対象のポインタが _adjusted_pointers にプッシュされる)

2. 次に, oopDesc::adjust_pointers() でポインタの修正処理を行う.

   (この際に, oopDesc::adjust_pointers() 内で MarkSweep::track_adjusted_pointer() が呼び出され, 
   実際に処理したポインタが _adjusted_pointers から取り除かれる)

3. その後に, MarkSweep::check_interior_pointers() で _adjusted_pointers が空かどうかをチェックする
   (処理対象が正しければ空になっているはず)


```cpp
    ((cite: hotspot/src/share/vm/memory/space.cpp))
    void Space::adjust_pointers() {
    ...
          VALIDATE_MARK_SWEEP_ONLY(MarkSweep::track_interior_pointers(oop(q)));
          // point all the oops to the new location
          size_t size = oop(q)->adjust_pointers();
          VALIDATE_MARK_SWEEP_ONLY(MarkSweep::check_interior_pointers());
```

#### 使用箇所(where its instances are used)
MarkSweep::track_interior_pointers() 内で(のみ)使用されている.

(なお, このクラスは (#ifdef VALIDATE_MARK_SWEEP 時であることに加えて) 
ValidateMarkSweep オプションが指定されている場合にしか使用されない)


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    void MarkSweep::track_interior_pointers(oop obj) {
      if (ValidateMarkSweep) {
        _adjusted_pointers->clear();
        _pointer_tracking = true;
    
        AdjusterTracker checker;
        obj->oop_iterate(&checker);
      }
    }
```

### 内部構造(Internal structure)
行う処理は, MarkSweep::check_adjust_pointer() を呼び出して, 
対象のポインタを _adjusted_pointers 内にプッシュするだけ.

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
      void do_oop(oop* o)       { MarkSweep::check_adjust_pointer(o); }
      void do_oop(narrowOop* o) { MarkSweep::check_adjust_pointer(o); }
```

```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp))
    void MarkSweep::check_adjust_pointer(void* p) {
      _adjusted_pointers->push(p);
    }
```

### 備考(Notes)
VALIDATE_MARK_SWEEP マクロ定数は以下のように #define される.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/shared/markSweep.hpp))
    // If VALIDATE_MARK_SWEEP is defined, the -XX:+ValidateMarkSweep flag will
    // be operational, and will provide slow but comprehensive self-checks within
    // the GC.  This is not enabled by default in product or release builds,
    // since the extra call to track_adjusted_pointer() in _adjust_pointer()
    // would be too much overhead, and would disturb performance measurement.
    // However, debug builds are sometimes way too slow to run GC tests!
    #ifdef ASSERT
    #define VALIDATE_MARK_SWEEP 1
    #endif
```




### 詳細(Details)
See: [here](../doxygen/classAdjusterTracker.html) for details

---
