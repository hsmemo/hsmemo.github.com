---
layout: default
title: ConcurrentMarkThread クラス (ConcurrentMarkThread, 及びその補助クラス(CMCheckpointRootsInitialClosure, CMCheckpointRootsFinalClosure, CMCleanUp))
---
[Top](../index.html)

#### ConcurrentMarkThread クラス (ConcurrentMarkThread, 及びその補助クラス(CMCheckpointRootsInitialClosure, CMCheckpointRootsFinalClosure, CMCleanUp))

これらは, G1CollectedHeap 使用時における Garbage Collection 処理用の補助クラス.
より具体的に言うと, Java プログラムの実行と並行(concurrent)に Marking 処理を行うクラス (See: [here](no2935d4w.html) for details).


### クラス一覧(class list)

  * [ConcurrentMarkThread](#nowh5I6kWN)
  * [CMCheckpointRootsInitialClosure](#noOpv-VYIC)
  * [CMCheckpointRootsFinalClosure](#noCTQVbJE2)
  * [CMCleanUp](#no3BSdpH-H)


---
## <a name="nowh5I6kWN" id="nowh5I6kWN">ConcurrentMarkThread</a>

### 概要(Summary)
G1GC アルゴリズムを補佐するスレッドクラス(ConcurrentGCThreadクラス).
このクラスは, Concurrent marking 処理を行う (See: [here](no2935d4w.html) for details).

(なおコメントには, CMS からコピーしてきたものでまだ作成中(under construction), みたいなことが書いてあるが...)


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.hpp))
    // The Concurrent Mark GC Thread (could be several in the future).
    // This is copied from the Concurrent Mark Sweep GC Thread
    // Still under construction.
```


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.hpp))
    class ConcurrentMarkThread: public ConcurrentGCThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 ConcurrentMark オブジェクトの _cmThread フィールドに(のみ)格納されている
(「各」と言っても1つしかいないが...).

#### 生成箇所(where its instances are created)
ConcurrentMark::ConcurrentMark() 内で(のみ)生成されている.

### 備考(Notes)
ConcurrentMarkThread オブジェクト自体は 1インスタンスしか存在しないが,
実際の処理は WorkGang (や G1RefProcTaskExecutor) 経由でマルチスレッド化される (See: [here](no2935d4w.html) for details).




### 詳細(Details)
See: [here](../doxygen/classConcurrentMarkThread.html) for details

---
## <a name="noOpv-VYIC" id="noOpv-VYIC">CMCheckpointRootsInitialClosure</a>

### 概要(Summary)
ConcurrentMarkThread クラス内で使用される補助クラス.

Initial marking pause 処理を行うための Closure クラス (See: [here](no2935d4w.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.cpp))
    class CMCheckpointRootsInitialClosure: public VoidClosure {
```

### 使われ方(Usage)
ConcurrentMarkThread::run() 内で(のみ)使用されている (See: [here](no2935d4w.html) for details).

なお, 使用する際は VM_CGC_Operation と併せて使用する (See: VM_CGC_Operation).




### 詳細(Details)
See: [here](../doxygen/classCMCheckpointRootsInitialClosure.html) for details

---
## <a name="noCTQVbJE2" id="noCTQVbJE2">CMCheckpointRootsFinalClosure</a>

### 概要(Summary)
ConcurrentMarkThread クラス内で使用される補助クラス.

Final marking pause 処理を行うための Closure クラス (See: [here](no2935d4w.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.cpp))
    class CMCheckpointRootsFinalClosure: public VoidClosure {
```

### 使われ方(Usage)
ConcurrentMarkThread::run() 内で(のみ)使用されている (See: [here](no2935d4w.html) for details).

なお, 使用する際は VM_CGC_Operation と併せて使用する (See: VM_CGC_Operation).




### 詳細(Details)
See: [here](../doxygen/classCMCheckpointRootsFinalClosure.html) for details

---
## <a name="no3BSdpH-H" id="no3BSdpH-H">CMCleanUp</a>

### 概要(Summary)
ConcurrentMarkThread クラス内で使用される補助クラス.

Live Data Counting & Cleanup 処理を行うための Closure クラス (See: [here](no2935d4w.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/g1/concurrentMarkThread.cpp))
    class CMCleanUp: public VoidClosure {
```

### 使われ方(Usage)
ConcurrentMarkThread::run() 内で(のみ)使用されている (See: [here](no2935d4w.html) for details).

なお, 使用する際は VM_CGC_Operation と併せて使用する (See: VM_CGC_Operation).




### 詳細(Details)
See: [here](../doxygen/classCMCleanUp.html) for details

---
