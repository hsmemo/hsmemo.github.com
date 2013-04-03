---
layout: default
title: WorkGang に関するクラス (AbstractGangTask, AbstractGangTaskWOopQueues, AbstractWorkGang, WorkData, WorkGang, GangWorker, FlexibleWorkGang, WorkGangBarrierSync, SubTasksDone, SequentialSubTasksDone, FreeIdSet)
---
[Top](../index.html)

#### WorkGang に関するクラス (AbstractGangTask, AbstractGangTaskWOopQueues, AbstractWorkGang, WorkData, WorkGang, GangWorker, FlexibleWorkGang, WorkGangBarrierSync, SubTasksDone, SequentialSubTasksDone, FreeIdSet)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, GC 処理をマルチスレッド化するためのクラス (See: [here](no28916ecK.html) for details).


### クラス一覧(class list)

  * [AbstractGangTask](#no2D2HjAMC)
  * [AbstractGangTaskWOopQueues](#nogEGfq_DP)
  * [AbstractWorkGang](#noHOja2VmB)
  * [WorkData](#notq5O6bDT)
  * [WorkGang](#noUHmF7do8)
  * [GangWorker](#noerQKil1H)
  * [FlexibleWorkGang](#notrrz7PVO)
  * [WorkGangBarrierSync](#norp6vhKSO)
  * [SubTasksDone](#noQO6YoLgn)
  * [SequentialSubTasksDone](#noo8zU4PV4)
  * [FreeIdSet](#noPXwRloRF)


---
## <a name="no2D2HjAMC" id="no2D2HjAMC">AbstractGangTask</a>

### 概要(Summary)
全ての GangTask クラスの基底クラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // An abstract task to be worked on by a gang.
    // You subclass this to supply your own work() method
    class AbstractGangTask VALUE_OBJ_CLASS_SPEC {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
      // The abstract work method.
      // The argument tells you which member of the gang you are.
      virtual void work(int i) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classAbstractGangTask.html) for details

---
## <a name="nogEGfq_DP" id="nogEGfq_DP">AbstractGangTaskWOopQueues</a>

### 概要(Summary)
特殊な AbstractGangTask クラス (See: [here](no28916ecK.html) for details).

通常の AbstractGangTask に加えて, 
OopTaskQueueSet と ParallelTaskTerminator を内部に格納している.
(何のため?? #TODO)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    class AbstractGangTaskWOopQueues : public AbstractGangTask {
```

(現状では CMS 専用のクラスである模様)

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.

### 使われ方(Usage)
CMSRefProcTaskProxy クラスのスーパークラスとして(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classAbstractGangTaskWOopQueues.html) for details

---
## <a name="noHOja2VmB" id="noHOja2VmB">AbstractWorkGang</a>

### 概要(Summary)
GangWorker スレッド全体を管理するクラスの基底クラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // Class AbstractWorkGang:
    // An abstract class representing a gang of workers.
    // You subclass this to supply an implementation of run_task().
    class AbstractWorkGang: public CHeapObj {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
      // Run a task, returns when the task is done (or terminated).
      virtual void run_task(AbstractGangTask* task) = 0;
```




### 詳細(Details)
See: [here](../doxygen/classAbstractWorkGang.html) for details

---
## <a name="notq5O6bDT" id="notq5O6bDT">WorkData</a>

### 概要(Summary)
GangWorker スレッドと WorkGang オブジェクト (GangWorker スレッドの取りまとめ役) の間で
GangTask 情報を受け渡す際に使用される補助クラス.

(より具体的に言うと, AbstractWorkGang::internal_worker_poll() と GangWorker::loop() で使用される補助クラス.
 各 GangWorker スレッドが取りまとめ役である WorkGang から次の GangTask を取得する際に使用される.)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    class WorkData: public StackObj {
```

### 内部構造(Internal structure)
単なる構造体のようなクラス.

内部には以下のフィールド(のみ)を含む
(そして, メソッドはこれらのフィールドへのアクセサメソッドのみ).

(コメントにも「struct でもよかったけどアクセサメソッドが欲しかったのでクラスにした」と書かれるほどシンプル)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
      // This would be a struct, but I want accessor methods.
    private:
      bool              _terminate;
      AbstractGangTask* _task;
      int               _sequence_number;
```




### 詳細(Details)
See: [here](../doxygen/classWorkData.html) for details

---
## <a name="noUHmF7do8" id="noUHmF7do8">WorkGang</a>

### 概要(Summary)
AbstractWorkGang クラスのサブクラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // Class WorkGang:
    class WorkGang: public AbstractWorkGang {
```

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.




### 詳細(Details)
See: [here](../doxygen/classWorkGang.html) for details

---
## <a name="noerQKil1H" id="noerQKil1H">GangWorker</a>

### 概要(Summary)
WorkGang を用いた並列 GC において, 実際の GC 処理を行う WorkerThread クラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // Class GangWorker:
    //   Several instances of this class run in parallel as workers for a gang.
    class GangWorker: public WorkerThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
AbstractWorkGang オブジェクトの _gang_workers フィールドに(のみ)格納されている.

(正確には, このフィールドは GangWorker の配列を格納するフィールド.
この中に, 使用される全ての GangWorker オブジェクトが格納されている)

(なお, 実際には AbstractWorkGang は abstract class であり,
サブクラスの FlexibleWorkGang 内の同名のフィールドにのみ格納されている模様)

#### 生成箇所(where its instances are created)
WorkGang::allocate_worker() というファクトリメソッドが用意されており, その中で(のみ)生成されている.
そして, このファクトリメソッドは, 現在は以下のパスで(のみ)呼び出されている.

```
* SharedHeap オブジェクトの初期化時

  SharedHeap::SharedHeap()
  -> WorkGang::initialize_workers()
     -> WorkGang::allocate_worker()

* G1GC の ConcurrentMark の初期化時

  ConcurrentMark::ConcurrentMark()
  -> WorkGang::initialize_workers()
     -> WorkGang::allocate_worker()
```




### 詳細(Details)
See: [here](../doxygen/classGangWorker.html) for details

---
## <a name="notrrz7PVO" id="notrrz7PVO">FlexibleWorkGang</a>

### 概要(Summary)
WorkGang クラスの具象サブクラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    class FlexibleWorkGang: public WorkGang {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* SharedHeap オブジェクトの _workers フィールド
* ConcurrentMark オブジェクトの _parallel_workers フィールド  (<= G1GC 用)

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* SharedHeap::SharedHeap()
* ConcurrentMark::ConcurrentMark()  (<= G1GC 用)




### 詳細(Details)
See: [here](../doxygen/classFlexibleWorkGang.html) for details

---
## <a name="norp6vhKSO" id="norp6vhKSO">WorkGangBarrierSync</a>

### 概要(Summary)
GangWorker スレッド間でバリア同期を取るためのクラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // A class that acts as a synchronisation barrier. Workers enter
    // the barrier and must wait until all other workers have entered
    // before any of them may leave.
    
    class WorkGangBarrierSync : public StackObj {
```

(現状では G1GC 専用のクラスである模様)

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. 予め WorkGangBarrierSync::set_n_workers() を呼んで, バリア同期を取る GangWorker スレッドの数をセットしておく.
2. 同期点に到達した GangWorker スレッドは WorkGangBarrierSync::enter() を呼び出す.
   (その GangWorker スレッドは全ての GangWorker スレッドが WorkGangBarrierSync::enter() を呼び出すまでブロックされる)

#### インスタンスの格納場所(where its instances are stored)
ConcurrentMark オブジェクトの
_first_overflow_barrier_sync フィールドおよび
_second_overflow_barrier_sync フィールドに(のみ)格納されている.




### 詳細(Details)
See: [here](../doxygen/classWorkGangBarrierSync.html) for details

---
## <a name="noQO6YoLgn" id="noQO6YoLgn">SubTasksDone</a>

### 概要(Summary)
複数の GangWorker スレッド間で, (ある程度大きな単位で) 負荷分散を行うためのクラス (See: [here](no28916ecK.html) for details).


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // A class to manage claiming of subtasks within a group of tasks.  The
    // subtasks will be identified by integer indices, usually elements of an
    // enumeration type.
    
    class SubTasksDone: public CHeapObj {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. あらかじめ, 分散したい各処理には, それぞれ unique な int 値を割り振っておく.

2. SubTasksDone の中身は int 配列になっている.

   配列中の値が 1 であれば該当の処理に誰かが着手済みであることを示し,
   逆に配列中の値が 0 であれば誰も手をつけていないことを示す.

3. 配列中の値は, SubTasksDone::is_task_claimed() メソッドで
   (指定した箇所が 0 であれば) アトミックに 1 に変更することができる (既に 1 であれば変更はできない).

   SubTasksDone::is_task_claimed() が false を返せば, 自分がその処理を取得できたことになる.

4. なお, 処理が終わった GangWorker は, 最後に SubTasksDone::all_tasks_completed() を呼び出すことになっている模様.

   (SubTasksDone オブジェクトは SubTasksDone::all_tasks_completed() が呼び出された回数をカウントしている.
   回数がスレッド数に等しくなれば, 全処理が終了したとして SubTasksDone オブジェクトは初期状態(全ての種別の処理が未処理の状態)に戻る.
   初期状態に戻す処理は SubTasksDone::clear() を呼び出すことで行われる.)




### 詳細(Details)
See: [here](../doxygen/classSubTasksDone.html) for details

---
## <a name="noo8zU4PV4" id="noo8zU4PV4">SequentialSubTasksDone</a>

### 概要(Summary)
特殊な SubTasksDone クラス (See: [here](no28916ecK.html) for details).

SubTasksDone クラスと異なり, GangWorker スレッドは自分が取得する処理を指定できない.
代わりに, 現在残っている処理の中から「一番番号が小さいもの」が勝手に割り振られて返される.
(この挙動は, 処理にあらかじめ番号が振れない場合に便利, とのこと.
 例えば, Remembered Set のスキャン処理等をスレッド数に応じた数で動的に分割したい, 等).

(なお, SubTasksDone と違ってこちらは StackObject だが SubTasksDone もそうしては?? とのコメントも...)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // As above, but for sequential tasks, i.e. instead of claiming
    // sub-tasks from a set (possibly an enumeration), claim sub-tasks
    // in sequential order. This is ideal for claiming dynamically
    // partitioned tasks (like striding in the parallel remembered
    // set scanning). Note that unlike the above class this is
    // a stack object - is there any reason for it not to be?
    
    class SequentialSubTasksDone : public StackObj {
```

### 内部構造(Internal structure)
SubTasksDone::is_task_claimed() では取得したい処理を引数で指定できるが,
SequentialSubTasksDone::is_task_claimed() では引数が参照渡しになっており,
逆に割り当てられた処理がこの引数経由で返される.

(処理は番号の小さいものから順に割り振られていく. 処理が全部割り振られてしまうと返値として true が返される.)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
      // Returns false if the next task in the sequence is unclaimed,
      // and ensures that it is claimed. Will set t to be the index
      // of the claimed task in the sequence. Will return true if
      // the task cannot be claimed and there are none left to claim.
      bool is_task_claimed(int& t);
```




### 詳細(Details)
See: [here](../doxygen/classSequentialSubTasksDone.html) for details

---
## <a name="noPXwRloRF" id="noPXwRloRF">FreeIdSet</a>

### 概要(Summary)
並列 GC 処理で各 GC スレッドに割り当てられる ID を管理するクラス.

(?? 現状では G1GC の DirtyCardQueueSet 内でしか使われていない模様... #TODO)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
    // Represents a set of free small integer ids.
    class FreeIdSet {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. FreeIdSet::claim_par_id() を呼ぶと, 使われていない ID が1つリターンされる.
2. 処理が終わったら, FreeIdSet::release_par_id() を呼んで ID を返却する.

#### インスタンスの格納場所(where its instances are stored)
(現状では G1GC 専用のクラスである模様)

DirtyCardQueueSet オブジェクトの _free_ids フィールドに(のみ)格納されている.

(なお, ここに格納されているオブジェクトへのポインタは FreeIdSet クラスの _sets フィールドにも格納されている)

#### 生成箇所(where its instances are created)
DirtyCardQueueSet::initialize() 内で(のみ)生成されている.

#### 使用箇所(where its instances are used)
DirtyCardQueueSet::mut_process_buffer() 内で(のみ)使用されている.

### 内部構造(Internal structure)
内部では, 未使用の ID を線形リスト状にして管理している.

(このフリーリストは, ids フィールドの int 配列と _hd フィールドの int 値で構成される.
_hd フィールドがフリーリストの先頭 index を保持しており, 
ids フィールドの配列要素はそれぞれリストの次の要素の index を格納している)


```
    ((cite: hotspot/src/share/vm/utilities/workgroup.hpp))
      int* _ids;
      int _hd;
```




### 詳細(Details)
See: [here](../doxygen/classFreeIdSet.html) for details

---
