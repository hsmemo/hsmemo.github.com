---
layout: default
title: CompileBroker クラス関連のクラス (CompileTask, CompilerCounters, CompileQueue, CompileTaskWrapper, CompileBroker)
---
[Top](../index.html)

#### CompileBroker クラス関連のクラス (CompileTask, CompilerCounters, CompileQueue, CompileTaskWrapper, CompileBroker)

これらは, JIT コンパイル処理を行う CompilerThread にコンパイル要求を伝えるためのクラス (See: [here](no7882MiN.html) for details).


### クラス一覧(class list)

  * [CompileBroker](#no-XzCIY00)
  * [CompileTask](#norIp-kiA8)
  * [CompileQueue](#nojZBuEZDH)
  * [CompileTaskWrapper](#noaXgfHng5)
  * [CompilerCounters](#nogYqiBFSL)


---
## <a name="no-XzCIY00" id="no-XzCIY00">CompileBroker</a>

### 概要(Summary)
JIT コンパイル要求に対する broker(仲介役) 的なクラス (より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

コンパイル要求はまず CompileBroker に伝えられ, そこから実際に処理を行う CompilerThread に渡される.

(なお, 実際に使用する JIT コンパイラの種類は CompileBroker によって隠蔽されている.
 そのため JIT コンパイル要求を出すコードからは C1, C2, Shark の違いは見えない.
 どの場合も CompilerBroker に要求を出すだけになる)

また, CompileBroker クラスはコンパイラスレッドを作成する機能も提供している (make_compiler_thread, init_compiler_threads).


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
    // Compilation
    //
    // The broker for all compilation requests.
    class CompileBroker: AllStatic {
```

### 内部構造(Internal structure)
このクラスの CompileBroker::compile_method() メソッドが JIT コンパイル要求を出す処理のエントリポイントになっている.
様々な処理からこのメソッドが呼び出されることで JIT コンパイルが開始される.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
      static nmethod* compile_method(methodHandle method,
                                     int osr_bci,
                                     int comp_level,
                                     methodHandle hot_method,
                                     int hot_count,
                                     const char* comment, TRAPS);
```




### 詳細(Details)
See: [here](../doxygen/classCompileBroker.html) for details

---
## <a name="norIp-kiA8" id="norIp-kiA8">CompileTask</a>

### 概要(Summary)
「JIT コンパイル要求」を表すためのクラス.

CompilerBroker に届けられたコンパイル要求は, CompileTask オブジェクトとして CompilerThread に送られる.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
    // CompileTask
    //
    // An entry in the compile queue.  It represents a pending or current
    // compilation.
    class CompileTask : public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classCompileTask.html) for details

---
## <a name="nojZBuEZDH" id="nojZBuEZDH">CompileQueue</a>

### 概要(Summary)
CompileTask オブジェクトを詰めるためのキュー.

CompilerBroker に届けられたコンパイル要求は, このキューを介して CompilerThread に伝えられる.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
    // CompileQueue
    //
    // A list of CompileTasks.
    class CompileQueue : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

(C1 用と C2 用の 2本が用意されている. Tiered でなければどちらか一方だけ/Tiered なら両方が使用される)

* CompilerBroker クラスの _c1_method_queue フィールド (static フィールド)
  
  (C1 用)

* CompilerBroker クラスの _c2_method_queue フィールド (static フィールド)
  
  (C2/Shark 用)

(なお CompilerThread 側には, CompilerThread の生成時に使用する CompilerQueue が引数として渡されている)


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
      static CompileQueue* _c2_method_queue;
      static CompileQueue* _c1_method_queue;
```

#### 生成箇所(where its instances are created)
CompileBroker::init_compiler_threads() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classCompileQueue.html) for details

---
## <a name="noaXgfHng5" id="noaXgfHng5">CompileTaskWrapper</a>

### 概要(Summary)
CompileTask クラス用のユーティリティ・クラス.

ソースコード中のあるスコープの間だけ,
CompilerThread に CompileTask をセットするための一時オブジェクト(StackObjクラス).

(また, CompileLog を用いたログ出力の開始/終了処理も行っているほか, 
 デストラクタ内では CompileTask へのロック待ちがいた場合, 完了したことを notifyAll() で通知したりもしている)


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
    // CompileTaskWrapper
    //
    // Assign this task to the current thread.  Deallocate the task
    // when the compilation is complete.
    class CompileTaskWrapper : StackObj {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* CompileBroker::compiler_thread_loop()
* AdvancedThresholdPolicy::select_task()

### 内部構造(Internal structure)
(コンストラクタでカレントスレッドに指定の CompileTask がセットされ, デストラクタで終了アドレスが記録される)
 

```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.cpp))
    CompileTaskWrapper::CompileTaskWrapper(CompileTask* task) {
      CompilerThread* thread = CompilerThread::current();
      thread->set_task(task);
      CompileLog*     log  = thread->log();
      if (log != NULL)  task->log_task_start(log);
    }
    
    CompileTaskWrapper::~CompileTaskWrapper() {
      CompilerThread* thread = CompilerThread::current();
      CompileTask* task = thread->task();
      CompileLog*  log  = thread->log();
      if (log != NULL)  task->log_task_done(log);
      thread->set_task(NULL);
      task->set_code_handle(NULL);
      DEBUG_ONLY(thread->set_env((ciEnv*)badAddress));
      if (task->is_blocking()) {
        MutexLocker notifier(task->lock(), thread);
        task->mark_complete();
        // Notify the waiting thread that the compilation has completed.
        task->lock()->notify_all();
      } else {
        task->mark_complete();
    
        // By convention, the compiling thread is responsible for
        // recycling a non-blocking CompileTask.
        CompileBroker::free_task(task);
      }
    }
```




### 詳細(Details)
See: [here](../doxygen/classCompileTaskWrapper.html) for details

---
## <a name="nogYqiBFSL" id="nogYqiBFSL">CompilerCounters</a>

### 概要(Summary)
JIT コンパイル処理に関する統計情報を溜めていくためのパフォーマンスカウンタ. 
内部的には PerfData を用いた記録を行う.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
    // CompilerCounters
    //
    // Per Compiler Performance Counters.
    //
    class CompilerCounters : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompilerThread オブジェクトの _counters フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
CompileBroker::init_compiler_threads() 内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.cpp))
    void CompileBroker::init_compiler_threads(int c1_compiler_count, int c2_compiler_count) {
    ...
      for (int i = 0; i < c2_compiler_count; i++) {
        // Create a name for our thread.
        sprintf(name_buffer, "C2 CompilerThread%d", i);
        CompilerCounters* counters = new CompilerCounters("compilerThread", i, CHECK);
        CompilerThread* new_thread = make_compiler_thread(name_buffer, _c2_method_queue, counters, CHECK);
        _method_threads->append(new_thread);
      }
    
      for (int i = c2_compiler_count; i < compiler_count; i++) {
        // Create a name for our thread.
        sprintf(name_buffer, "C1 CompilerThread%d", i);
        CompilerCounters* counters = new CompilerCounters("compilerThread", i, CHECK);
        CompilerThread* new_thread = make_compiler_thread(name_buffer, _c1_method_queue, counters, CHECK);
        _method_threads->append(new_thread);
      }
```

#### 使用箇所(where its instances are used)
CompileBroker::compiler_thread_loop() 内で参照されている (#TODO 他の使用箇所)

### 内部構造(Internal structure)
内部には, 以下の PerfCounter を備えている.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.hpp))
        char _current_method[cmname_buffer_length];
        PerfStringVariable* _perf_current_method;
    
        int  _compile_type;
        PerfVariable* _perf_compile_type;
    
        PerfCounter* _perf_time;
        PerfCounter* _perf_compiles;
```




### 詳細(Details)
See: [here](../doxygen/classCompilerCounters.html) for details

---
