---
layout: default
title: MemProfiler クラス (MemProfiler, 及びその補助クラス(MemProfilerTask))
---
[Top](../index.html)

#### MemProfiler クラス (MemProfiler, 及びその補助クラス(MemProfilerTask))



### クラス一覧(class list)

  * [MemProfiler](#noyEWe-mZ2)
  * [MemProfilerTask](#nov3eeIIfW)


---
## <a name="noyEWe-mZ2" id="noyEWe-mZ2">MemProfiler</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (関連する develop オプションが指定されている場合にのみ使用される) (See: MemProfiling).

定期的にメモリ使用量をログファイル(mprofile.log)に出力するための機能を納めた名前空間(AllStatic クラス).


```
    ((cite: hotspot/src/share/vm/runtime/memprofiler.hpp))
    // Prints periodic memory usage trace of HotSpot VM
```


```
    ((cite: hotspot/src/share/vm/runtime/memprofiler.hpp))
    class MemProfiler : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* HotSpot の起動時

  Threads::create_vm() の内で MemProfiler::engage() が呼ばれて処理が開始される.

* HotSpot の実行中

  MemProfilerTask::task() で, 定期間隔で MemProfiler::do_trace() が呼ばれてログファイルへの出力が行われる.

* HotSpot の終了時

  print_statistics() の内で MemProfiler::disengage() が呼ばれて後片付けが行われる.




### 詳細(Details)
See: [here](../doxygen/classMemProfiler.html) for details

---
## <a name="nov3eeIIfW" id="nov3eeIIfW">MemProfilerTask</a>

### 概要(Summary)
MemProfiler クラス内で使用される補助クラス.

デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

定期間隔でメモリ使用量をログファイル(mprofile.log)に出力するためのクラス(PeriodicTaskクラス).


```
    ((cite: hotspot/src/share/vm/runtime/memprofiler.cpp))
    #ifndef PRODUCT
    
    // --------------------------------------------------------
    // MemProfilerTask
    
    class MemProfilerTask : public PeriodicTask {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
MemProfiler クラスの _task フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
MemProfiler::engage() 内で(のみ)生成されている.

### 内部構造(Internal structure)
定期間隔で MemProfiler::do_trace() を呼び出しているだけ.

#### 参考(for your information): MemProfilerTask::task()
See: [here](no3420PGg.html) for details



### 詳細(Details)
See: [here](../doxygen/classMemProfilerTask.html) for details

---
