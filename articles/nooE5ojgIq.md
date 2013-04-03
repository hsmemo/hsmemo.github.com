---
layout: default
title: YieldingFlexibleWorkGang に関するクラス (YieldingFlexibleGangWorker, FlexibleGangTask, YieldingFlexibleGangTask, YieldingFlexibleWorkGang)
---
[Top](../index.html)

#### YieldingFlexibleWorkGang に関するクラス (YieldingFlexibleGangWorker, FlexibleGangTask, YieldingFlexibleGangTask, YieldingFlexibleWorkGang)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, GC 処理をマルチスレッド化するためのクラス (See: [here](no28916ecK.html) for details).



### クラス一覧(class list)

  * [YieldingFlexibleGangWorker](#noVQlONyOQ)
  * [FlexibleGangTask](#noieK6ZtWG)
  * [YieldingFlexibleGangTask](#no2mcwwp_U)
  * [YieldingFlexibleWorkGang](#noNZUdY4pc)


---
## <a name="noVQlONyOQ" id="noVQlONyOQ">YieldingFlexibleGangWorker</a>

### 概要(Summary)
特殊な GangWorker クラス (See: [here](no28916ecK.html) for details).

(通常の GangWorker クラスとの違いは...#TODO)


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
    // Class YieldingFlexibleGangWorker:
    //   Several instances of this class run in parallel as workers for a gang.
    class YieldingFlexibleGangWorker: public GangWorker {
```

(現状では CMS 専用のクラスである模様)

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
AbstractWorkGang オブジェクトの _gang_workers フィールドに(のみ)格納されている.

(正確には, このフィールドは GangWorker の配列を格納するフィールド.
この中に, 使用される全ての YieldingFlexibleGangWorker オブジェクトが格納されている)

(なお, 実際には AbstractWorkGang は abstract class であり,
サブクラスの YieldingFlexibleWorkGang 内の同名のフィールドにのみ格納されている模様)

#### 生成箇所(where its instances are created)
YieldingFlexibleWorkGang::allocate_worker() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
CMSCollector::CMSCollector()
-> WorkGang::initialize_workers()
   -> YieldingFlexibleWorkGang::allocate_worker()
```




### 詳細(Details)
See: [here](../doxygen/classYieldingFlexibleGangWorker.html) for details

---
## <a name="noieK6ZtWG" id="noieK6ZtWG">FlexibleGangTask</a>

### 概要(Summary)
AbstractGangTask クラスのサブクラスの1つ.


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
    class FlexibleGangTask: public AbstractGangTask {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
      // The abstract work method.
      // The argument tells you which member of the gang you are.
      virtual void work(int i) = 0;
```

### 内部構造(Internal structure)
AbstractGangTask と比べると, 以下の2つのフィールドが増えている点が異なる
(ついでに, このフィールドへのアクセサメソッドも追加されている).


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
      int _actual_size;                      // size of gang obtained
    protected:
      int _requested_size;                   // size of gang requested
```




### 詳細(Details)
See: [here](../doxygen/classFlexibleGangTask.html) for details

---
## <a name="no2mcwwp_U" id="no2mcwwp_U">YieldingFlexibleGangTask</a>

### 概要(Summary)
FlexibleGangTask クラスのサブクラス (なお, 現在はこのクラスが唯一のサブクラス).

途中で yield() したり abort() したりできる GangTask, らしい (#TODO).


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
    // An abstract task to be worked on by a flexible work gang,
    // and where the workers will periodically yield, usually
    // in response to some condition that is signalled by means
    // that are specific to the task at hand.
    // You subclass this to supply your own work() method.
    // A second feature of this kind of work gang is that
    // it allows for the signalling of certain exceptional
    // conditions that may be encountered during the performance
    // of the task and that may require the task at hand to be
    // `aborted' forthwith. Finally, these gangs are `flexible'
    // in that they can operate at partial capacity with some
    // gang workers waiting on the bench; in other words, the
    // size of the active worker pool can flex (up to an apriori
    // maximum) in response to task requests at certain points.
    // The last part (the flexible part) has not yet been fully
    // fleshed out and is a work in progress.
    class YieldingFlexibleGangTask: public FlexibleGangTask {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classYieldingFlexibleGangTask.html) for details

---
## <a name="noNZUdY4pc" id="noNZUdY4pc">YieldingFlexibleWorkGang</a>

### 概要(Summary)
特殊な WorkGang クラス

Worker Thread として YieldingGangWorkers クラスを使用しており, 
途中で yield する機能を提供する, らしい (#TODO).


```
    ((cite: hotspot/src/share/vm/utilities/yieldingWorkgroup.hpp))
    // Class YieldingWorkGang: A subclass of WorkGang.
    // In particular, a YieldingWorkGang is made up of
    // YieldingGangWorkers, and provides infrastructure
    // supporting yielding to the "GangOverseer",
    // being the thread that orchestrates the WorkGang via run_task().
    class YieldingFlexibleWorkGang: public FlexibleWorkGang {
```

(現状では CMS 専用のクラスである模様)

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
CMSCollector オブジェクトの _conc_workers フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
CMSCollector::CMSCollector() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classYieldingFlexibleWorkGang.html) for details

---
