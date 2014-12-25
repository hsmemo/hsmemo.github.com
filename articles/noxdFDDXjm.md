---
layout: default
title: Closure クラス関連のクラス (Closure, OopClosure, ObjectClosure, BoolObjectClosure, ObjectToOopClosure, UpwardsObjectClosure, ObjectClosureCareful, BlkClosure, BlkClosureCareful, SpaceClosure, CompactibleSpaceClosure, CodeBlobClosure, MarkingCodeBlobClosure, MarkingCodeBlobClosure::MarkScope, CodeBlobToOopClosure, MonitorClosure, VoidClosure, YieldClosure, SerializeOopClosure, SymbolClosure, RememberKlassesChecker)
---
[Top](../index.html)

#### Closure クラス関連のクラス (Closure, OopClosure, ObjectClosure, BoolObjectClosure, ObjectToOopClosure, UpwardsObjectClosure, ObjectClosureCareful, BlkClosure, BlkClosureCareful, SpaceClosure, CompactibleSpaceClosure, CodeBlobClosure, MarkingCodeBlobClosure, MarkingCodeBlobClosure::MarkScope, CodeBlobToOopClosure, MonitorClosure, VoidClosure, YieldClosure, SerializeOopClosure, SymbolClosure, RememberKlassesChecker)

これらは, メモリ管理用 (主に Garbage Collection 処理用) の補助クラス (See: [here](no2114GzS.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // The following classes are C++ `closures` for iterating over objects, roots and spaces
```

### 概要(Summary)
GC 処理等ではメモリ中のポインタ(oopDesc)全体に対して iterate 処理を行う必要がある.
こういった処理のために, HotSpot には関数型言語の高階関数のような (あるいは内部イテレータのような) フレームワークが用意されている.
この "Closure" を用いることで, オブジェクトを辿る処理(closure を受け取る高階関数のようなもの)と,
実際の各オブジェクトに対する処理(closure部分)を分離している.

  * Closure クラス
    
    関数型言語の「クロージャー(Closure)」を模したクラス.
    
    ポインタ値に対して行う処理はこちらに記述する.

    (なお Closure クラス自体は abstract な基底クラスであり, 
    実行したい処理に応じた各種のサブクラスが存在する)

  * Closure を呼び出す関数 (e.g. Threads::oops_do(), frame::oops_do(), etc)
    
    高階関数的な代物.
    
    GC の対象になるポインタ(oop)を含むクラスは, 
    その oop に Closure を適用する関数 (<= map() 関数のようなもの) を提供している.

(ちなみに, "Closure" という考え方はポインタ以外の処理でも利用されていたりする. 
 例えば「ビットマップ中のビットを処理する Closure」みたいなものも沢山存在する)

(なお, "Closure" 的に働くクラスが全て Closure クラスのサブクラスになっている訳ではない. 
 名前も実際の働きも Closure なのに StackObj のサブクラスだったりするものも沢山存在している)

(なお, (C++ なので当然だが) 自由変数を勝手に取り込んでくれる訳ではないので, 自分でクロージャー変換する必要がある)

### 備考(Notes)
Closure クラスには以下のようなサブクラスが用意されている.

(なお, Closure を適用するために呼び出すメソッドはサブクラス毎に名前が異なるので注意.
 do_oop() だったり do_object() だったり do_object_b() だったり do_code_blob() だったりする)

        Closure (abstract)
          OopClosure (abstract)
            OopsInGenClosure (abstract)
            ... (いっぱいサブクラスがある)
          ObjectClosure (abstract)
            ... (いっぱいサブクラスがある)
          CodeBlobClosure (abstract)
            MarkingCodeBlobClosure (abstract)
              CodeBlobToOopClosure
            MarkActivationClosure

  * OopClosure

    (ポインタ値を保持している)フィールドに対する処理を表す Closure
    処理は void do_oop(oop* o) メソッドに定義される.

    (処理対象が「ポインタ値を保持するフィールド(= oop へのポインタ)」なのでポインタ値の fixup 等も行える.
     ポインタを再帰的に辿っていく, といった処理で使用される模様.)

  * ObjectClosure

    オブジェクトに対する処理を表す Closure.
    処理は void do_object(oop obj) メソッドに定義される.

    (ある領域内のオブジェクトを全て辿る, といった処理で使用される模様.
    こういった処理の場合は, dead オブジェクトの場合そのオブジェクトを指しているフィールドはないので, OopClosure ではうまくいかない)

  * CodeBlobClosure

    CodeBlob に対する処理を表す Closure.
    処理は void do_code_blob(CodeBlob* cb) メソッドに定義される.

  * ...(#TODO)



### クラス一覧(class list)

  * [Closure](#noG-XlGgxT)
  * [OopClosure](#noWXudxLIG)
  * [ObjectClosure](#nogcJ3gKMz)
  * [BoolObjectClosure](#noCliM0AAc)
  * [ObjectToOopClosure](#noGDkP9Jbq)
  * [UpwardsObjectClosure](#nogJdKTbAQ)
  * [ObjectClosureCareful](#no1G3l5egT)
  * [BlkClosure](#no6SAgLQbW)
  * [BlkClosureCareful](#noDNfibiwn)
  * [SpaceClosure](#noqCVA5BdE)
  * [CompactibleSpaceClosure](#noMCeUAe4g)
  * [CodeBlobClosure](#noInFjG28h)
  * [MarkingCodeBlobClosure](#noPnz5_POl)
  * [MarkingCodeBlobClosure::MarkScope](#nobF7go6i6)
  * [CodeBlobToOopClosure](#no7TMIlGMP)
  * [MonitorClosure](#novC1aXuqz)
  * [VoidClosure](#no2EgatPBS)
  * [YieldClosure](#no5IZBsXoe)
  * [SerializeOopClosure](#noKW_OEjHt)
  * [SymbolClosure](#noOHBeCQb-)
  * [RememberKlassesChecker](#nohKAcDpEE)


---
## <a name="noG-XlGgxT" id="noG-XlGgxT">Closure</a>

### 概要(Summary)
全ての Closure クラスの基底クラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // Closure provides abortability.
    
    class Closure : public StackObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classClosure.html) for details

---
## <a name="noWXudxLIG" id="noWXudxLIG">OopClosure</a>

### 概要(Summary)
oop へのポインタに対して何らかの処理を行う Closure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // OopClosure is used for iterating through roots (oop*)
    
    class OopClosure : public Closure {
```

### 使われ方(Usage)
oop* (又は narrowOop*) を処理する do_oop() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual void do_oop(oop* o) = 0;
      virtual void do_oop_v(oop* o) { do_oop(o); }
      virtual void do_oop(narrowOop* o) = 0;
      virtual void do_oop_v(narrowOop* o) { do_oop(o); }
```




### 詳細(Details)
See: [here](../doxygen/classOopClosure.html) for details

---
## <a name="nogcJ3gKMz" id="nogcJ3gKMz">ObjectClosure</a>

### 概要(Summary)
oop に対して何らかの処理を行う Closure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // ObjectClosure is used for iterating through an object space
    
    class ObjectClosure : public Closure {
```

### 使われ方(Usage)
oop を処理する do_object() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Called for each object.
      virtual void do_object(oop obj) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classObjectClosure.html) for details

---
## <a name="noCliM0AAc" id="noCliM0AAc">BoolObjectClosure</a>

### 概要(Summary)
oop に対して何らかの判定を行う(= bool を返す) Closure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    class BoolObjectClosure : public ObjectClosure {
```

### 使われ方(Usage)
oop を判定する do_object_b() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual bool do_object_b(oop obj) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classBoolObjectClosure.html) for details

---
## <a name="noGDkP9Jbq" id="noGDkP9Jbq">ObjectToOopClosure</a>

### 概要(Summary)
他の OopClosure と組み合わせて使用される Closure クラス (より正確には ObjectClosure クラス).

指定された OopClosure をある oop 内の全てのポインタフィールドに対して適用する.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // Applies an oop closure to all ref fields in objects iterated over in an
    // object iteration.
    class ObjectToOopClosure: public ObjectClosure {
```

### 使われ方(Usage)
コンストラクタ引数として OopClosure オブジェクトを受け取る.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      ObjectToOopClosure(OopClosure* cl) : _cl(cl) {}
```

実行されると, 引数で渡された oop に対して,
コンストラクタ引数で指定された OopClosure を実行する.

#### 参考(for your information): ObjectToOopClosure::do_object()
See: [here](no7882rQZ.html) for details



### 詳細(Details)
See: [here](../doxygen/classObjectToOopClosure.html) for details

---
## <a name="nogJdKTbAQ" id="nogJdKTbAQ">UpwardsObjectClosure</a>

### 概要(Summary)
何らかの情報を記憶しておく機能を備えた BoolObjectClosure クラス (の基底クラス)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // A version of ObjectClosure with "memory" (see _previous_address below)
    class UpwardsObjectClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
情報を記録しておくための _previous_address というフィールドを備えている.
このフィールドには, set_previous() メソッドと previous() メソッドでアクセスできる
(逆に, 明示的にアクセスしない限り, このフィールドは特に使われることはない).

(なお, このフィールドは「前回に呼び出された際の情報」を記録しておくのに使われる模様. 
少なくとも, 現状でこのクラスの唯一のサブクラスである ScanMarkedObjectsAgainClosure 
内ではそう使われている. (See: ScanMarkedObjectsAgainClosure))


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      HeapWord* _previous_address;
    ...
      UpwardsObjectClosure() : _previous_address(NULL) { }
      void set_previous(HeapWord* addr) { _previous_address = addr; }
      HeapWord* previous()              { return _previous_address; }
```

メソッドとしては oop を判定する do_object_bm() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // A return value of "true" can be used by the caller to decide
      // if this object's end should *NOT* be recorded in
      // _previous_address above.
      virtual bool do_object_bm(oop obj, MemRegion mr) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classUpwardsObjectClosure.html) for details

---
## <a name="no1G3l5egT" id="no1G3l5egT">ObjectClosureCareful</a>

### 概要(Summary)
処理対象の中に uninitialized な oop があるかもしれない場合用の ObjectClosure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // A version of ObjectClosure that is expected to be robust
    // in the face of possibly uninitialized objects.
    class ObjectClosureCareful : public ObjectClosure {
```

### 使われ方(Usage)
oop を処理する do_object_careful() メソッド及び do_object_careful_m() メソッドを備えている
(これらは uninitialized な oop に適用されると 0 をリターンすることになっている模様).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual size_t do_object_careful_m(oop p, MemRegion mr) = 0;
      virtual size_t do_object_careful(oop p) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classObjectClosureCareful.html) for details

---
## <a name="no6SAgLQbW" id="no6SAgLQbW">BlkClosure</a>

### 概要(Summary)
(CMS 用語で言うところの) "block" に対して何らかの処理を行う Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // Blk closure (abstract class)
    class BlkClosure : public StackObj {
```

なお, このクラス(のサブクラス)は CMS の処理の中で使用される.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // The following are used in CompactibleFreeListSpace and
    // ConcurrentMarkSweepGeneration.
```

### 使われ方(Usage)
block (を表す HeapWord*) を処理する do_blk() メソッドを備えている
(このメソッドは処理した block の大きさを表す size_t 値をリターンする).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual size_t do_blk(HeapWord* addr) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classBlkClosure.html) for details

---
## <a name="noDNfibiwn" id="noDNfibiwn">BlkClosureCareful</a>

### 概要(Summary)
処理対象の中に uninitialized な oop があるかもしれない場合用の BlkClosure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // A version of BlkClosure that is expected to be robust
    // in the face of possibly uninitialized objects.
    class BlkClosureCareful : public BlkClosure {
```

なお, このクラス(のサブクラス)は CMS の処理の中で使用される.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // The following are used in CompactibleFreeListSpace and
    // ConcurrentMarkSweepGeneration.
```

### 使われ方(Usage)
block (を表す HeapWord*) を処理する do_blk_careful() メソッドを備えている
(uninitialized な oop に適用された場合は... #TODO).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual size_t do_blk_careful(HeapWord* addr) = 0;
```

なお, BlkClosure から引き継いだ do_blk() メソッドは使用禁止にされている.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      size_t do_blk(HeapWord* addr) {
        guarantee(false, "call do_blk_careful instead");
        return 0;
      }
```




### 詳細(Details)
See: [here](../doxygen/classBlkClosureCareful.html) for details

---
## <a name="noqCVA5BdE" id="noqCVA5BdE">SpaceClosure</a>

### 概要(Summary)
Space に対して何らかの処理を行う Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // SpaceClosure is used for iterating over spaces
    ...
    class SpaceClosure : public StackObj {
```

### 使われ方(Usage)
Space* を処理する do_space() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Called for each space
      virtual void do_space(Space* s) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classSpaceClosure.html) for details

---
## <a name="noMCeUAe4g" id="noMCeUAe4g">CompactibleSpaceClosure</a>

### 概要(Summary)
CompactibleSpace に対して何らかの処理を行う Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    class CompactibleSpaceClosure : public StackObj {
```

### 使われ方(Usage)
CompactibleSpace* を処理する do_space() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Called for each compactible space
      virtual void do_space(CompactibleSpace* s) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classCompactibleSpaceClosure.html) for details

---
## <a name="noInFjG28h" id="noInFjG28h">CodeBlobClosure</a>

### 概要(Summary)
CodeBlob に対して何らかの処理を行う Closure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // CodeBlobClosure is used for iterating through code blobs
    // in the code cache or on thread stacks
    
    class CodeBlobClosure : public Closure {
```

### 使われ方(Usage)
CodeBlob* を処理する do_code_blob() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Called for each code blob.
      virtual void do_code_blob(CodeBlob* cb) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classCodeBlobClosure.html) for details

---
## <a name="noPnz5_POl" id="noPnz5_POl">MarkingCodeBlobClosure</a>

### 概要(Summary)
同じ CodeBlob に対しては最大1回しか処理を行わない CodeBlobClosure クラス (の基底クラス).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    class MarkingCodeBlobClosure : public CodeBlobClosure {
```

### 使われ方(Usage)
CodeBlob* を処理する do_newly_marked_nmethod() メソッドを備えている
(実際に使用されるときには, まず do_code_blob() メソッドが呼び出され, そこで 1回目か ２回目以降かの判定が行われた後, 
1回目であれば do_newly_marked_nmethod() メソッドが呼び出される模様).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Called for each code blob, but at most once per unique blob.
      virtual void do_newly_marked_nmethod(nmethod* nm) = 0;
    
      virtual void do_code_blob(CodeBlob* cb);
```

なお, このクラス(のサブクラス)は MarkingCodeBlobClosure::MarkScope (のサブクラス) とセットで使用される
(See: MarkingCodeBlobClosure::MarkScope).

#### 参考(for your information): MarkingCodeBlobClosure::do_code_blob()
See: [here](no78825bm.html) for details



### 詳細(Details)
See: [here](../doxygen/classMarkingCodeBlobClosure.html) for details

---
## <a name="nobF7go6i6" id="nobF7go6i6">MarkingCodeBlobClosure::MarkScope</a>

### 概要(Summary)
MarkingCodeBlobClosure クラスとセットで使用するための補助クラス (の基底クラス).

CodeBlob 内の GC 処理で使用される
(より正確に言うと, nmethod の GC 処理のための補助クラス.
GC 処理の前後で, nmethod を処理するための補助構造の準備/後始末を行う).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      class MarkScope : public StackObj {
```

### 使われ方(Usage)
コード中で局所変数として生成される.

コンストラクタで nmethod の GC 処理用の前準備を行い (nmethod::oops_do_marking_prologue() の呼び出し),
デストラクタで nmethod 用の後始末を行う (nmethod::oops_do_marking_epilogue() の呼び出し).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.cpp))
    MarkingCodeBlobClosure::MarkScope::MarkScope(bool activate)
      : _active(activate)
    {
      if (_active)  nmethod::oops_do_marking_prologue();
    }
    
    MarkingCodeBlobClosure::MarkScope::~MarkScope() {
      if (_active)  nmethod::oops_do_marking_epilogue();
    }
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

また, このクラス(のサブクラス)は MarkingCodeBlobClosure (のサブクラス) とセットで使用される
(See: MarkingCodeBlobClosure).

### 内部構造(Internal structure)
nmethod クラスは, nmethod::_oops_do_mark_nmethods という static フィールドを持っている.
そして, 同じ nmethod を複数回処理したくない場合には, 
処理した nmethod オブジェクトをこのフィールドに線形リスト状につないでいく.


```cpp
    ((cite: hotspot/src/share/vm/code/nmethod.cpp))
    #define NMETHOD_SENTINEL ((nmethod*)badAddress)
    
    nmethod* volatile nmethod::_oops_do_mark_nmethods;
```

この処理は, MarkingCodeBlobClosure::MarkScope (のサブクラス) が呼び出す
nmethod::oops_do_marking_prologue() メソッドと nmethod::oops_do_marking_epilogue() メソッド,
及び MarkingCodeBlobClosure (のサブクラス) が呼び出す
nmethod::test_set_oops_do_mark() メソッドで実現されている.

#### 参考(for your information): nmethod::oops_do_marking_prologue()
See: [here](no7882gPV.html) for details
#### 参考(for your information): nmethod::test_set_oops_do_mark()
See: [here](no7882TFP.html) for details
#### 参考(for your information): nmethod::oops_do_marking_epilogue()
See: [here](no7882tZb.html) for details



### 詳細(Details)
See: [here](../doxygen/classMarkingCodeBlobClosure_1_1MarkScope.html) for details

---
## <a name="no7TMIlGMP" id="no7TMIlGMP">CodeBlobToOopClosure</a>

### 概要(Summary)
CodeBlob に対して処理を行う Closure クラス

(<= CodeBlobClosure (や MarkingCodeBlobClosure) には他にサブクラスがいないので, 
このクラスが実際の CodeBlob の処理で使用される模様 #TODO).

他の OopClosure と組み合わせて使用される Closure クラス.
指定の CodeBlob に対して指定の OopClosure を適用する処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // Applies an oop closure to all ref fields in code blobs
    // iterated over in an object iteration.
    class CodeBlobToOopClosure: public MarkingCodeBlobClosure {
```

### 使われ方(Usage)
コンストラクタ引数として OopClosure オブジェクトを受け取る
(また, marking 処理を使用するかどうかを示す do_marking 引数も受け取る).


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      CodeBlobToOopClosure(OopClosure* cl, bool do_marking)
```

メソッドとしては, CodeBlob* を処理する do_code_blob() メソッド, 及び do_newly_marked_nmethod() メソッドを備えている.

* do_code_blob() メソッドの方は, CodeBlob に対してコンストラクタ引数で受け取った OopClosure を適用するだけ.

* do_newly_marked_nmethod() メソッドの方は, do_marking コンストラクタ引数の値によって少し挙動が変わる.
  * do_marking が true の場合は, MarkingCodeBlobClosure::do_code_blob() を呼んで marking 処理を用いた呼び出しを行う.
  * do_marking が false の場合は, CodeBlob に対してコンストラクタ引数で受け取った OopClosure を適用するだけ.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual void do_newly_marked_nmethod(nmethod* cb);
        // = { cb->oops_do(_cl); }
      virtual void do_code_blob(CodeBlob* cb);
        // = { if (_do_marking)  super::do_code_blob(cb); else cb->oops_do(_cl); }
```

#### 参考(for your information): CodeBlobToOopClosure::do_newly_marked_nmethod()
See: [here](no7882F6B.html) for details
#### 参考(for your information): CodeBlobToOopClosure::do_code_blob()
See: [here](no7882SEI.html) for details



### 詳細(Details)
See: [here](../doxygen/classCodeBlobToOopClosure.html) for details

---
## <a name="novC1aXuqz" id="novC1aXuqz">MonitorClosure</a>

### 概要(Summary)
ObjectMonitor に対して何らかの処理を行う Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // MonitorClosure is used for iterating over monitors in the monitors cache
    ...
    class MonitorClosure : public StackObj {
```

### 使われ方(Usage)
ObjectMonitor* を処理する do_monitor() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // called for each monitor in cache
      virtual void do_monitor(ObjectMonitor* m) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classMonitorClosure.html) for details

---
## <a name="no2EgatPBS" id="no2EgatPBS">VoidClosure</a>

### 概要(Summary)
引数を持たない Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // A closure that is applied without any arguments.
    class VoidClosure : public StackObj {
```

### 使われ方(Usage)
0-arity の do_void() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

(なおコメントによると, 
このクラスの do_void() は純粋仮想関数にしたかったがよく分からない理由により上手くいかなかった, 
とのこと)

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // I would have liked to declare this a pure virtual, but that breaks
      // in mysterious ways, for unknown reasons.
      virtual void do_void();
```

#### 参考(for your information): VoidClosure::do_void()
See: [here](no7882T3m.html) for details



### 詳細(Details)
See: [here](../doxygen/classVoidClosure.html) for details

---
## <a name="no5IZBsXoe" id="no5IZBsXoe">YieldClosure</a>

### 概要(Summary)
途中で中断を挟みながら処理を行うための Closure クラス (の基底クラス).

(現状ではサブクラスが CMSPrecleanRefsYieldClosure しかいないため, CMS の GC 処理専用のクラス)

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // YieldClosure is intended for use by iteration loops
    // to incrementalize their work, allowing interleaving
    // of an interruptable task so as to allow other
    // threads to run (which may not otherwise be able to access
    // exclusive resources, for instance). Additionally, the
    // closure also allows for aborting an ongoing iteration
    // by means of checking the return value from the polling
    // call.
    class YieldClosure : public StackObj {
```

### 使われ方(Usage)
何らかの処理を行う should_return() メソッドを備える.
should_return() メソッドは, 呼び出し元が処理を止めてリターンすべきかどうかを示す bool 値を返す.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
       virtual bool should_return() = 0;
```




### 詳細(Details)
See: [here](../doxygen/classYieldClosure.html) for details

---
## <a name="noKW_OEjHt" id="noKW_OEjHt">SerializeOopClosure</a>

### 概要(Summary)
serialize されたデータを読み書きするための Closure クラス (の基底クラス).

(現状ではサブクラスが WriteClosure と ReadClosure しかいないため, CDS の shared archive 処理専用のクラス)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    // Abstract closure for serializing data (read or write).
    
    class SerializeOopClosure : public OopClosure {
```

### 使われ方(Usage)
serialize されたデータを処理するための以下のようなメソッドが定義されている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      // Return bool indicating whether closure implements read or write.
      virtual bool reading() const = 0;
    
      // Read/write the int pointed to by i.
      virtual void do_int(int* i) = 0;
    
      // Read/write the size_t pointed to by i.
      virtual void do_size_t(size_t* i) = 0;
    
      // Read/write the void pointer pointed to by p.
      virtual void do_ptr(void** p) = 0;
    
      // Read/write the HeapWord pointer pointed to be p.
      virtual void do_ptr(HeapWord** p) = 0;
    
      // Read/write the region specified.
      virtual void do_region(u_char* start, size_t size) = 0;
    
      // Check/write the tag.  If reading, then compare the tag against
      // the passed in value and fail is they don't match.  This allows
      // for verification that sections of the serialized data are of the
      // correct length.
      virtual void do_tag(int tag) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classSerializeOopClosure.html) for details

---
## <a name="noOHBeCQb-" id="noOHBeCQb-">SymbolClosure</a>

### 概要(Summary)
Symbol に対して何らかの処理を行う Closure クラス (の基底クラス).

(なお, このクラスは Closure クラスのサブクラスではなく, StackObj のサブクラスになっている)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    class SymbolClosure : public StackObj {
```

### 使われ方(Usage)
Symbol** を処理する do_symbol() メソッドを備えている.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual void do_symbol(Symbol**) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classSymbolClosure.html) for details

---
## <a name="nohKAcDpEE" id="nohKAcDpEE">RememberKlassesChecker</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか定義されない).

GC 処理中のクラス・アンローディングが行われる可能性がある箇所で, 
「ここの処理では should_remember_klasses() や remember_klass() メソッドをオーバーライドした OopClosure を使わなければいけない」
ということをコード上に明示したい場合に使用される.

(See: OopClosure::must_remember_klasses(), OopClosure::should_remember_klasses(), OopClosure::remember_klass())

(なお, コンストラクタ引数に false を渡した場合, 
そのスコープ内だけ RememberKlassesChecker の働きを停止させる, という効果がある.
これは CMS で使用されている模様)


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
    #ifdef ASSERT
    // This class is used to flag phases of a collection that
    // can unload classes and which should override the
    // should_remember_klasses() and remember_klass() of OopClosure.
    // The _must_remember_klasses is set in the contructor and restored
    // in the destructor.  _must_remember_klasses is checked in assertions
    // in the OopClosure implementations of should_remember_klasses() and
    // remember_klass() and the expectation is that the OopClosure
    // implementation should not be in use if _must_remember_klasses is set.
    // Instances of RememberKlassesChecker can be place in
    // marking phases of collections which can do class unloading.
    // RememberKlassesChecker can be passed "false" to turn off checking.
    // It is used by CMS when CMS yields to a different collector.
    class RememberKlassesChecker: StackObj {
```

### 使われ方(Usage)
コード中で RememberKlassesChecker 型の局所変数を宣言するだけ.

なお, コンストラクタ引数として true を渡すと上記のようなチェックが有効になる.
逆にコンストラクタ引数に false を渡した場合, そのスコープ内だけ RememberKlassesChecker のチェックが停止される.

### 内部構造(Internal structure)
OopClosure クラスには, デバッグ用の OopClosure::_must_remember_klasses というフィールドがある.
RememberKlassesChecker はこのフィールドの値をセット／リセットする.

(なお, このフィールドのアクセサメソッドは, 
OopClosure::set_must_remember_klasses() 及び OopClosure::must_remember_klasses())


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.cpp))
    #ifdef ASSERT
    bool OopClosure::_must_remember_klasses = false;
    #endif
```

コンストラクタで OopClosure::set_must_remember_klasses() を呼んで _must_remember_klasses を変更し, 
デストラクタで (同じく OopClosure::set_must_remember_klasses() を呼んで) 元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      RememberKlassesChecker(bool checking_on) : _saved_state(false),
        _do_check(true) {
        // The ClassUnloading unloading flag affects the collectors except
        // for CMS.
        // CMS unloads classes if CMSClassUnloadingEnabled is true or
        // if ExplicitGCInvokesConcurrentAndUnloadsClasses is true and
        // the current collection is an explicit collection.  Turning
        // on the checking in general for
        // ExplicitGCInvokesConcurrentAndUnloadsClasses and
        // UseConcMarkSweepGC should not lead to false positives.
        _do_check =
          ClassUnloading && !UseConcMarkSweepGC ||
          CMSClassUnloadingEnabled && UseConcMarkSweepGC ||
          ExplicitGCInvokesConcurrentAndUnloadsClasses && UseConcMarkSweepGC;
        if (_do_check) {
          _saved_state = OopClosure::must_remember_klasses();
          OopClosure::set_must_remember_klasses(checking_on);
        }
      }
      ~RememberKlassesChecker() {
        if (_do_check) {
          OopClosure::set_must_remember_klasses(_saved_state);
        }
      }
```

そして, この OopClosure::_must_remember_klasses フィールドの値は
OopClosure::should_remember_klasses() 内でチェックされている
(値が false でなければ assert failure になる).

(なお, 上のコメントに反して, OopClosure::remember_klass() の方では何のチェックも行われていないようだが... #TODO)


#### 参考(for your information): OopClosure::set_must_remember_klasses()
See: [here](no7882H1b.html) for details
#### 参考(for your information): OopClosure::must_remember_klasses()
See: [here](no7882U_h.html) for details
#### 参考(for your information): OopClosure::should_remember_klasses()
See: [here](no7882TMD.html) for details
#### 参考(for your information): OopClosure::remember_klass()

```cpp
    ((cite: hotspot/src/share/vm/memory/iterator.hpp))
      virtual void remember_klass(Klass* k) { /* do nothing */ }
```




### 詳細(Details)
See: [here](../doxygen/classRememberKlassesChecker.html) for details

---
