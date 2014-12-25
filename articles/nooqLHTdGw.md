---
layout: default
title: PSScavenge クラス関連のクラス (PSScavenge, PSScavengeRootsClosure, 及びそれらの補助クラス(PSIsAliveClosure, PSKeepAliveClosure, PSEvacuateFollowersClosure, PSPromotionFailedClosure, PSRefProcTaskProxy, PSRefEnqueueTaskProxy, PSRefProcTaskExecutor))
---
[Top](../index.html)

#### PSScavenge クラス関連のクラス (PSScavenge, PSScavengeRootsClosure, 及びそれらの補助クラス(PSIsAliveClosure, PSKeepAliveClosure, PSEvacuateFollowersClosure, PSPromotionFailedClosure, PSRefProcTaskProxy, PSRefEnqueueTaskProxy, PSRefProcTaskExecutor))

これらは, ParallelScavengeHeap の Minor GC 処理
("Parallel Scavenge" 処理) で使用される補助クラス (See: [here](no289165Un.html) for details).



### クラス一覧(class list)

  * [PSScavenge](#noOrfCY4G6)
  * [PSScavengeRootsClosure](#nocvQk9vhu)
  * [PSIsAliveClosure](#noMW9rUEgT)
  * [PSKeepAliveClosure](#not-B0gyma)
  * [PSEvacuateFollowersClosure](#noMGrM8voH)
  * [PSPromotionFailedClosure](#nofC5qIW9g)
  * [PSRefProcTaskProxy](#no4is0QU63)
  * [PSRefEnqueueTaskProxy](#notfOT6ChP)
  * [PSRefProcTaskExecutor](#noid0GwGlF)


---
## <a name="noOrfCY4G6" id="noOrfCY4G6">PSScavenge</a>

### 概要(Summary)
ParallelScavengeHeap の Minor GC 処理を行うクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.hpp))
    class PSScavenge: AllStatic {
```

### 使われ方(Usage)
Minor GC 処理はこのクラスの PSScavenge::invoke() メソッドが呼び出されることで実行される (See: [here](no289165Un.html) for details).




### 詳細(Details)
See: [here](../doxygen/classPSScavenge.html) for details

---
## <a name="nocvQk9vhu" id="nocvQk9vhu">PSScavengeRootsClosure</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(Closureクラス).

まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.inline.hpp))
    class PSScavengeRootsClosure: public OopClosure {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている
(より正確には, PSScavenge::invoke_no_policy() 内と,
そこから呼び出される ScavengeRootsTask::do_it() と ThreadRootsTask::do_it() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSScavengeRootsClosure.html) for details

---
## <a name="noMW9rUEgT" id="noMW9rUEgT">PSIsAliveClosure</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(Closureクラス).

GC 時の参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.
PSIsAliveClosure::do_object_b() メソッドが呼ばれると, 
処理対象のオブジェクトが生きているかどうかを返す.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSIsAliveClosure: public BoolObjectClosure {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
PSScavenge クラスの _is_alive_closure フィールドに格納されているほか,
PSRefProcTaskProxy::do_it() 内で局所変数として生成されている.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.hpp))
    class PSScavenge: AllStatic {
    ...
      static PSIsAliveClosure    _is_alive_closure;     // Closure used for reference processing
```

#### 使用箇所(where its instances are used)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている
(より正確には, PSScavenge::invoke_no_policy() 内と,
そこから呼び出される PSRefProcTaskProxy::do_it() 内で使用される補助クラス).




### 詳細(Details)
See: [here](../doxygen/classPSIsAliveClosure.html) for details

---
## <a name="not-B0gyma" id="not-B0gyma">PSKeepAliveClosure</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(Closureクラス).

GC 時の参照オブジェクト(java.lang.ref オブジェクト)の処理に用いられる Closure クラス.

まだコピーされていないオブジェクトに対して, コピー処理を行い, 
さらに元の場所にフォワーディングポインタを埋める処理を行う.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSKeepAliveClosure: public OopClosure {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている
(より正確には, PSScavenge::invoke_no_policy() 内と,
そこから呼び出される PSRefProcTaskProxy::do_it() 内で使用される補助クラス).

### 内部構造(Internal structure)
処理自体は PSScavengeRootsClosure とよく似ている. (See: PSScavengeRootsClosure)

ただし, このクラスの場合は To 領域のコピー済みのオブジェクトは処理対象にしないという点が異なる.




### 詳細(Details)
See: [here](../doxygen/classPSKeepAliveClosure.html) for details

---
## <a name="noMGrM8voH" id="noMGrM8voH">PSEvacuateFollowersClosure</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(Closureクラス).

処理したオブジェクトから辿れる範囲全てについて再帰的に処理を行うための Closure.


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSEvacuateFollowersClosure: public VoidClosure {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている
(より正確には, PSScavenge::invoke_no_policy() 内と,
そこから呼び出される PSRefProcTaskProxy::do_it() 内で使用される補助クラス).

現在は, GC 時の参照オブジェクト(java.lang.ref オブジェクト)の処理にのみ用いられている
(live だと分かった参照オブジェクトから再帰的に辿る処理に使用されているのみ).




### 詳細(Details)
See: [here](../doxygen/classPSEvacuateFollowersClosure.html) for details

---
## <a name="nofC5qIW9g" id="nofC5qIW9g">PSPromotionFailedClosure</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス(Closureクラス).

GC 処理が失敗した際に, 各オブジェクトの mark フィールドを初期状態に戻すための Closure
(GC 処理後には mark フィールドには forwarding pointer が埋められているため, それをクリアする).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSPromotionFailedClosure : public ObjectClosure {
```

### 使われ方(Usage)
PSScavenge::clean_up_failed_promotion() 内で(のみ)使用されている (See: [here](no289165Un.html) for details).




### 詳細(Details)
See: [here](../doxygen/classPSPromotionFailedClosure.html) for details

---
## <a name="no4is0QU63" id="no4is0QU63">PSRefProcTaskProxy</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス.

参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された 
AbstractRefProcTaskExecutor::ProcessTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::ProcessTask))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSRefProcTaskProxy: public GCTask {
```

### 使われ方(Usage)
PSRefProcTaskExecutor::execute(ProcessTask& task) 内で(のみ)使用されている (See: [here](no289165Un.html) and [here](no289169tf.html) for details).

### 内部構造(Internal structure)
参照オブジェクト処理のマルチスレッド化自体は PSRefProcTaskExecutor クラスが行っている.

各 PSRefProcTaskProxy オブジェクトには
PSRefProcTaskExecutor クラスによって
実行すべき AbstractRefProcTaskExecutor::ProcessTask オブジェクトが 1つ割り当てられるので, 単にそれを実行するだけ.

#### 参考(for your information): PSRefProcTaskProxy::do_it()
See: [here](no28916OMU.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSRefProcTaskProxy.html) for details

---
## <a name="notfOT6ChP" id="notfOT6ChP">PSRefEnqueueTaskProxy</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス.

参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するための補助クラス
(より具体的に言うと, コンストラクタ引数で渡された 
AbstractRefProcTaskExecutor::EnqueueTask オブジェクトを実行するためのクラス
(See: AbstractRefProcTaskExecutor::EnqueueTask))


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSRefEnqueueTaskProxy: public GCTask {
```

### 使われ方(Usage)
PSRefProcTaskExecutor::execute(EnqueueTask& task) 内で(のみ)使用されている (See: [here](no289165Un.html) and [here](no289169tf.html) for details).

### 内部構造(Internal structure)
参照オブジェクト処理のマルチスレッド化自体は PSRefProcTaskExecutor クラスが行っている.

各 PSRefEnqueueTaskProxy オブジェクトには
PSRefProcTaskExecutor クラスによって
実行すべき AbstractRefProcTaskExecutor::EnqueueTask オブジェクトが 1つ割り当てられるので, 単にそれを実行するだけ.

#### 参考(for your information): PSRefEnqueueTaskProxy::do_it()
See: [here](no28916bWa.html) for details



### 詳細(Details)
See: [here](../doxygen/classPSRefEnqueueTaskProxy.html) for details

---
## <a name="noid0GwGlF" id="noid0GwGlF">PSRefProcTaskExecutor</a>

### 概要(Summary)
ParallelScavengeHeap に対する Minor GC 処理で使用される補助クラス.

ParallelScavengeHeap に対する Minor GC 処理 ("Parallel Scavenge" 処理) で使用される AbstractRefProcTaskExecutor クラス.
(つまり, Parallel Scavenge 処理における参照オブジェクト(java.lang.ref オブジェクト)の処理をマルチスレッド化するためのクラス
 (See: AbstractRefProcTaskExecutor)).


```cpp
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/psScavenge.cpp))
    class PSRefProcTaskExecutor: public AbstractRefProcTaskExecutor {
```

### 使われ方(Usage)
PSScavenge::invoke_no_policy() 内で(のみ)使用されている (See: [here](no289165Un.html) and [here](no289169tf.html) for details).




### 詳細(Details)
See: [here](../doxygen/classPSRefProcTaskExecutor.html) for details

---
