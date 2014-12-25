---
layout: default
title: CompilationPolicy クラス関連のクラス (CompilationPolicy, NonTieredCompPolicy, SimpleCompPolicy, StackWalkCompPolicy, 及びそれらの補助クラス(CounterDecay))
---
[Top](../index.html)

#### CompilationPolicy クラス関連のクラス (CompilationPolicy, NonTieredCompPolicy, SimpleCompPolicy, StackWalkCompPolicy, 及びそれらの補助クラス(CounterDecay))

これらは, JIT Compiler 用のクラス.
より具体的に言うと, メソッドをコンパイルするかどうかの条件/閾値を管理するクラス
(See: [here](no3718SNC.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.hpp))
    // The CompilationPolicy selects which method (if any) should be compiled.
    // It also decides which methods must always be compiled (i.e., are never
    // interpreted).
```


### クラス一覧(class list)

  * [CompilationPolicy](#noZLKZ5g2q)
  * [NonTieredCompPolicy](#no_vuihGHe)
  * [SimpleCompPolicy](#noKBtdjj8Q)
  * [StackWalkCompPolicy](#noXPLmLhB9)
  * [CounterDecay](#nobGTbgENT)


---
## <a name="noZLKZ5g2q" id="noZLKZ5g2q">CompilationPolicy</a>

### 概要(Summary)
JIT Compiler 用のクラス.
メソッドをコンパイルするかどうかの条件/閾値を管理するクラス(の基底クラス).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス
(See: [here](no3718SNC.html) for details).


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.hpp))
    class CompilationPolicy : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classCompilationPolicy.html) for details

---
## <a name="no_vuihGHe" id="no_vuihGHe">NonTieredCompPolicy</a>

### 概要(Summary)
CompilationPolicy クラスのサブクラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.hpp))
    // A base class for baseline policies.
    class NonTieredCompPolicy : public CompilationPolicy {
```




### 詳細(Details)
See: [here](../doxygen/classNonTieredCompPolicy.html) for details

---
## <a name="noKBtdjj8Q" id="noKBtdjj8Q">SimpleCompPolicy</a>

### 概要(Summary)
NonTieredCompPolicy クラスの具象サブクラス
(See: [here](no3420bIr.html) for details).

なお, このクラスは CompilationPolicyChoice オプションが 0 の場合にのみ使用される
(そして Tiered Compilation 非使用時のデフォルトは 0). (See: compilationPolicy_init())


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.hpp))
    class SimpleCompPolicy : public NonTieredCompPolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CompilationPolicy クラスの _policy フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
compilationPolicy_init() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> compilationPolicy_init()
```




### 詳細(Details)
See: [here](../doxygen/classSimpleCompPolicy.html) for details

---
## <a name="noXPLmLhB9" id="noXPLmLhB9">StackWalkCompPolicy</a>

### 概要(Summary)
C2 JIT Compiler 用の補助クラス (#ifdef COMPILER2 時にしか定義されない).

C2 JIT Compiler 用の CompilationPolicy クラス(= メソッドをコンパイルするかどうかの条件/閾値を管理するクラス)
(See: [here](no3718SNC.html) and [here](no34200pY.html) for details).

なお, このクラスは CompilationPolicyChoice オプションが 1 の場合にのみ使用される. 
(See: compilationPolicy_init())


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.hpp))
    // StackWalkCompPolicy - existing C2 policy
    
    #ifdef COMPILER2
    class StackWalkCompPolicy : public NonTieredCompPolicy {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CompilationPolicy クラスの _policy フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
compilationPolicy_init() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
(HotSpot の起動時処理) (See: [here](no2114J7x.html) for details)
-> Threads::create_vm()
   -> init_globals()
      -> compilationPolicy_init()
```




### 詳細(Details)
See: [here](../doxygen/classStackWalkCompPolicy.html) for details

---
## <a name="nobGTbgENT" id="nobGTbgENT">CounterDecay</a>

### 概要(Summary)
NonTieredCompPolicy クラス内で使用される補助クラス.

ある程度の時間間隔で InvocationCounter の値を半減させていくためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).


```cpp
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.cpp))
    //
    // CounterDecay
    //
    // Interates through invocation counters and decrements them. This
    // is done at each safepoint.
    //
    class CounterDecay : public AllStatic {
```

### 使われ方(Usage)
NonTieredCompPolicy::do_safepoint_work() 内で(のみ)使用されている.

(なお, この関数は毎回の Safepoint 処理の先頭で呼び出されている (See: [here](no2935qaz.html) for details))

### 内部構造(Internal structure)
現状では, 約 CounterHalfLifeTime 秒ごとに各 InvocationCounter の値を半分にしている.

CounterDecay クラスの処理は, 約 CounterDecayMinIntervalLength ミリ秒ごとに実行され, 
一度に全クラス数の (CounterDecayMinIntervalLength * 1e-3 / CounterHalfLifeTime) *100% を処理する
(これにより約 CounterHalfLifeTime 秒で全クラスの処理が完了する).

#### 参考(for your information): NonTieredCompPolicy::do_safepoint_work()
See: [here](no3420xRn.html) for details
#### 参考(for your information): CounterDecay::is_decay_needed()
See: [here](no3420-bt.html) for details
#### 参考(for your information): CounterDecay::decay()
See: [here](no34209vC.html) for details
#### 参考(for your information): CounterDecay::do_method()
See: [here](no3420K6I.html) for details
#### 参考(for your information): InvocationCounter::decay()
See: [here](no3420Lmz.html) for details



### 詳細(Details)
See: [here](../doxygen/classCounterDecay.html) for details

---
