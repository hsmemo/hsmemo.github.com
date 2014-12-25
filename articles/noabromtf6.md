---
layout: default
title: PeriodicTask クラス 
---
[Top](../index.html)

#### PeriodicTask クラス 



---
## <a name="noQXnw0etv" id="noQXnw0etv">PeriodicTask</a>

### 概要(Summary)
定期間隔で何らかの処理を実行するクラスの基底クラス (See: [here](nohcAO37b3.html) for details).

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```cpp
    ((cite: hotspot/src/share/vm/runtime/task.hpp))
    // A PeriodicTask has the sole purpose of executing its task
    // function with regular intervals.
    // Usage:
    //   PeriodicTask pf(10);
    //   pf.enroll();
    //   ...
    //   pf.disenroll();
    
    class PeriodicTask: public CHeapObj {
```

### 使われ方(Usage)
使用する際には, task() メソッドをオーバーライドしたサブクラスを作ればいい.


```cpp
    ((cite: hotspot/src/share/vm/runtime/task.hpp))
      // The task to perform at each period
      virtual void task() = 0;
```

### 内部構造(Internal structure)
内部的には, WatcherThread によって定期間隔での実行を実現している.
各 PeriodicTask オブジェクトの task() メソッドは以下のパスで呼び出される (See: WatcherThread).

```
WatcherThread::run()
-> PeriodicTask::real_time_tick()
   -> PeriodicTask::execute_if_pending()
      -> PeriodicTask::task()
```




### 詳細(Details)
See: [here](../doxygen/classPeriodicTask.html) for details

---
