---
layout: default
title: CompileLog クラス 
---
[Top](../index.html)

#### CompileLog クラス 



---
## <a name="notSc_V2Gj" id="notSc_V2Gj">CompileLog</a>

### 概要(Summary)
トラブルシューティング用のクラス (関連する diagnostic オプションが指定されている場合にのみ使用される) (See: LogCompilation).

JIT コンパイル処理に関するログ出力を行うための xmlStream クラス.


```
    ((cite: hotspot/src/share/vm/compiler/compileLog.hpp))
    // CompileLog
    //
    // An open stream for logging information about activities in a
    // compiler thread.  There is exactly one per CompilerThread,
    // if the +LogCompilation switch is enabled.
    class CompileLog : public xmlStream {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 CompilerThread オブジェクトの _log フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
CompileBroker::init_compiler_thread_log() 内で(のみ)生成されている.


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.cpp))
    void CompileBroker::compiler_thread_loop() {
    ...
      // Open a log.
      if (LogCompilation) {
        init_compiler_thread_log();
      }
```

#### 使用箇所(where its instances are used)
実際のロギングの開始/終了処理は CompileTaskWrapper によって行われている (See: CompileTaskWrapper).


```
    ((cite: hotspot/src/share/vm/compiler/compileBroker.cpp))
    CompileTaskWrapper::CompileTaskWrapper(CompileTask* task) {
      CompilerThread* thread = CompilerThread::current();
      thread->set_task(task);
      CompileLog*     log  = thread->log();
      if (log != NULL)  task->log_task_start(log);
```




### 詳細(Details)
See: [here](../doxygen/classCompileLog.html) for details

---
