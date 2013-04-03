---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
  ~ThreadInVMfromJavaNoAsyncException()  {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadStateTransition::trans() を呼び出して, JavaThreadState を _thread_in_Java に変更する.
      (Safepoint 処理が開始されていた場合は, この中でブロックする)
      ---------------------------------------- -}

	    trans(_thread_in_vm, _thread_in_Java);

  {- -------------------------------------------
  (1) JavaThread::has_special_runtime_exit_condition() を呼んで
      処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックしておく.
      もし何か起きていたら, JavaThread::handle_special_runtime_exit_condition() を呼んで対処する.
  
      (なお, ThreadInVMfromJava とは異なり, 
       JavaThread::handle_special_runtime_exit_condition() は, 引数を false に指定して呼び出す.
       See: ThreadInVMfromJava::~ThreadInVMfromJava())
      ---------------------------------------- -}

	    // NOTE: We do not check for pending. async. exceptions.
	    // If we did and moved the pending async exception over into the
	    // pending exception field, we would need to deopt (currently C2
	    // only). However, to do so would require that we transition back
	    // to the _thread_in_vm state. Instead we postpone the handling of
	    // the async exception.
	
	    // Check for pending. suspends only.
	    if (_thread->has_special_runtime_exit_condition())
	      _thread->handle_special_runtime_exit_condition(false);
	  }
	
```


