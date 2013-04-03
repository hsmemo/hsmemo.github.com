---
layout: default
title: GCTaskThread クラス及びその補助クラス (GCTaskThread, GCTaskTimeStamp)
---
[Top](../index.html)

#### GCTaskThread クラス及びその補助クラス (GCTaskThread, GCTaskTimeStamp)

これらは, Garbage Collection 処理用の補助クラス.
より具体的に言うと, 実際の GC 処理を行う Thread クラス (See: [here](no24805iK.html) for details).


### クラス一覧(class list)

  * [GCTaskThread](#no_lGrkZ50)
  * [GCTaskTimeStamp](#nola7dIR0L)


---
## <a name="no_lGrkZ50" id="no_lGrkZ50">GCTaskThread</a>

### 概要(Summary)
ParallelScavenge GC において, 実際の GC 処理を行う WorkerThread クラス (See: [here](no24805iK.html) for details).


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp))
    class GCTaskThread : public WorkerThread {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
GCTaskManager クラスの _thread フィールドに(のみ)格納されている
(正確には, このフィールドは GCTaskThread の配列を格納するフィールド.
この中に, 使用される全ての GCTaskManager オブジェクトが格納されている)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskManager.hpp))
    class GCTaskManager : public CHeapObj {
    ...
      GCTaskThread**            _thread;            // Array of worker threads.
```

#### 生成箇所(where its instances are created)
GCTaskThread::create() というファクトリメソッドが用意されており, その中で(のみ)生成されている.

(そして, このファクトリメソッドは GCTaskManager::initialize() 内で(のみ)呼び出されている)

#### 参考(for your information): GCTaskThread::create()
See: [here](no7882DQl.html) for details
#### 参考(for your information): GCTaskManager::initialize()
See: [here](no7882Qar.html) for details



### 詳細(Details)
See: [here](../doxygen/classGCTaskThread.html) for details

---
## <a name="nola7dIR0L" id="nola7dIR0L">GCTaskTimeStamp</a>

### 概要(Summary)
保守運用機能のためのクラス (関連するオプションが指定されている場合にのみ使用される) (See: PrintGCTaskTimeStamps).

GCTaskThread クラス内で使用される補助クラス.
ParallelScavenge 関連の統計情報を格納するために使用される.


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp))
    class GCTaskTimeStamp : public CHeapObj
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
GCTaskThread クラスの _time_stamps インスタンスフィールドに(のみ)格納されている 

(正確には, このフィールドは GCTaskThread の配列を格納するフィールド. 
この中に GCTaskTimeStampEntries 個数分の GCTaskTimeStamp オブジェクトが格納されている)

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp))
      GCTaskTimeStamp* _time_stamps;
```

#### 生成箇所(where its instances are created)
GCTaskThread::GCTaskThread() 内で(のみ)生成されている.

(ただし, PrintGCTaskTimeStamps オプションが指定されている場合にしか生成されない)

#### 参考(for your information): GCTaskThread::GCTaskThread()
See: [here](no7882c4G.html) for details
#### 情報の記録箇所(where information is recorded)
GCTaskThread::run() 内で情報の記録が行われている.

(ただし, PrintGCTaskTimeStamps オプションが指定されている場合にしか記録されない)

GCTack 実行の前後で TimeStamp オブジェクトを使って時間を記録し, 
その時間と実行した GCTask の名前を記録している.
(なお, GCTaskTimeStampEntries 個数分の GCTaskTimeStamp オブジェクトを保持しているのでローテーションで使用している. 
次に使用する GCTaskTimeStamp オブジェクトは _time_stamp_index フィールドで管理している)


```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.cpp))
    void GCTaskThread::run() {
    ...
      TimeStamp timer;
    ...
          // In case the update is costly
          if (PrintGCTaskTimeStamps) {
            timer.update();
          }
    ...
          task->do_it(manager(), which());
    ...
          if (PrintGCTaskTimeStamps) {
            assert(_time_stamps != NULL, "Sanity (PrintGCTaskTimeStamps set late?)");
    
            timer.update();
    
            GCTaskTimeStamp* time_stamp = time_stamp_at(_time_stamp_index++);
    
            time_stamp->set_name(name);
            time_stamp->set_entry_time(entry_time);
            time_stamp->set_exit_time(timer.ticks());
          }
```

#### 情報の出力箇所(where the recorded information is output)
集めた結果は, GCTaskThread::print_task_time_stamps() で(のみ)使用されている (この関数で集めた結果が出力されている).

なお, この関数は PSParallelCompact::invoke_no_policy() や
PSScavenge::invoke_no_policy() 内から以下のように呼び出される.

```
PSParallelCompact::invoke_no_policy()
-> GCTaskManager::print_task_time_stamps()
   -> GCTaskThread::print_task_time_stamps()
```

```
PSScavenge::invoke_no_policy()
-> GCTaskManager::print_task_time_stamps()
   -> 同上
```

#### 参考(for your information): GCTaskManager::print_task_time_stamps()
See: [here](no7882drl.html) for details
#### 参考(for your information): GCTaskThread::print_task_time_stamps()
See: [here](no7882q1r.html) for details
### 内部構造(Internal structure)
内部には以下の3つのフィールド(のみ)を含む.
それぞれ, GC 処理が行われた時間(開始時間, 終了時間), 実行した GCTask 名, を記録する.

```
    ((cite: hotspot/src/share/vm/gc_implementation/parallelScavenge/gcTaskThread.hpp))
      jlong  _entry_time;
      jlong  _exit_time;
      char*  _name;
```




### 詳細(Details)
See: [here](../doxygen/classGCTaskTimeStamp.html) for details

---
