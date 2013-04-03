---
layout: default
title: MemoryService クラス関連のクラス (MemoryService, TraceMemoryManagerStats, 及びそれらの補助クラス(GcThreadCountClosure))
---
[Top](../index.html)

#### MemoryService クラス関連のクラス (MemoryService, TraceMemoryManagerStats, 及びそれらの補助クラス(GcThreadCountClosure))

これらは, Platform MXBean 機能のためのクラス.
より具体的に言うと, メモリ関係の Platform MXBean 機能へのアクセスを提供するブローカー的なクラス
(java.lang.management.MemoryPoolMXBean や java.lang.management.MemoryManagerMXBean から使用される).
(See: [here](no2114twV.html) for details)
(See: [here](no211477i.html) for details)


### クラス一覧(class list)

  * [MemoryService](#noCdw7Oyw-)
  * [TraceMemoryManagerStats](#noqzRr3gFv)
  * [GcThreadCountClosure](#noZsnLLXOG)


---
## <a name="noCdw7Oyw-" id="noCdw7Oyw-">MemoryService</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 用の Java クラスからのみ使用される)
(See: java.lang.management.MemoryPoolMXBean, java.lang.management.MemoryManagerMXBean).
(See: [here](no2114twV.html) and [here](no211477i.html) for details)

メモリ関係の Platform MXBean に関する機能を納めた名前空間(AllStatic クラス)

```
    ((cite: hotspot/src/share/vm/services/memoryService.hpp))
    // VM Monitoring and Management Support
    
    class MemoryService : public AllStatic {
```

各 MemoryPool オブジェクトのファクトリメソッドを提供しているほか,
全ての MemoryPool オブジェクトや MemoryManager オブジェクトにアクセスするための関数を提供している
(HotSpot 内で MemoryPool や MemoryManager オブジェクトを使いたくなった時には
 MemoryService クラスを介してこれらにアクセスする模様).




### 詳細(Details)
See: [here](../doxygen/classMemoryService.html) for details

---
## <a name="noqzRr3gFv" id="noqzRr3gFv">TraceMemoryManagerStats</a>

### 概要(Summary)
MemoryService クラスを簡単に利用するための補助クラス(StackObjクラス).

GC の開始時/終了時には MemoryService::gc_begin()/MemoryService::gc_end() を呼び出す必要があるが, 
この処理をソースコード上のスコープに合わせて自動で行うことができる.

(なお, MemoryService::gc_begin()/MemoryService::gc_end() は, 
JMM 用の統計情報を取得したり DTrace や JMM のフック処理を呼び出すための関数)


```
    ((cite: hotspot/src/share/vm/services/memoryService.hpp))
    class TraceMemoryManagerStats : public StackObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード中で TraceMemoryManagerStats 型の局所変数を宣言するだけ.

#### 使用箇所(where its instances are used)
各種 GC 処理中の処理を開始する地点に局所変数として宣言されている.

### 内部構造(Internal structure)
コンストラクタで MemoryService::gc_begin() を呼び出し, デストラクタで MemoryService::gc_end() を呼び出す.

#### 参考(for your information): TraceMemoryManagerStats::TraceMemoryManagerStats()
See: [here](no21140Fv.html) for details
#### 参考(for your information): TraceMemoryManagerStats::initialize()
See: [here](no2114BQ1.html) for details
#### 参考(for your information): MemoryService::gc_begin()
See: [here](no2114NuQ.html) for details
#### 参考(for your information): GCMemoryManager::gc_begin()
See: [here](no2114nCd.html) for details
#### 参考(for your information): TraceMemoryManagerStats::~TraceMemoryManagerStats()
See: [here](no2114AkK.html) for details
#### 参考(for your information): MemoryService::gc_end()
See: [here](no2114a4W.html) for details
#### 参考(for your information): GCMemoryManager::gc_end()
See: [here](no21140Mj.html) for details



### 詳細(Details)
See: [here](../doxygen/classTraceMemoryManagerStats.html) for details

---
## <a name="noZsnLLXOG" id="noZsnLLXOG">GcThreadCountClosure</a>

### 概要(Summary)
MemoryService クラス内で使用される補助クラス.

GC thread の合計数を数えるための Closure.


```
    ((cite: hotspot/src/share/vm/services/memoryService.cpp))
    class GcThreadCountClosure: public ThreadClosure {
```

### 使われ方(Usage)
MemoryService::set_universe_heap() 内で(のみ)使用されている.

### 内部構造(Internal structure)
GcThreadCountClosure::do_thread() ではカウンタをインクリメントしているだけ
(このメソッドが各 GC スレッドに対して一度ずつ呼び出されるので合計スレッド数が数えられる).


```
    ((cite: hotspot/src/share/vm/services/memoryService.cpp))
    void GcThreadCountClosure::do_thread(Thread* thread) {
      _count++;
    }
```




### 詳細(Details)
See: [here](../doxygen/classGcThreadCountClosure.html) for details

---
