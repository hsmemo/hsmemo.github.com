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
  ThreadToNativeFromVM(JavaThread *thread) : ThreadStateTransition(thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この処理では, VM の中から出て native code に遷移する. もし safepoint の途中であればブロックさせる)
      ---------------------------------------- -}

	    // We are leaving the VM at this point and going directly to native code.
	    // Block, if we are in the middle of a safepoint synchronization.

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(!thread->owns_locks(), "must release all locks when leaving VM");

  {- -------------------------------------------
  (1) JavaFrameAnchor::make_walkable() を呼んで, スタックフレームを辿れるようにしておく.
      ---------------------------------------- -}

	    thread->frame_anchor()->make_walkable(thread);

  {- -------------------------------------------
  (1) ThreadStateTransition::trans_and_fence() を呼んで, JavaThreadState を _thread_in_native に変更する.
      ---------------------------------------- -}

	    trans_and_fence(_thread_in_vm, _thread_in_native);

  {- -------------------------------------------
  (1) JavaThread::has_special_runtime_exit_condition() を呼んで
      処理対象のスレッドが例外を出していたりサスペンドされたりしていないかチェックしておく.
      もし何か起きていたら, JavaThread::handle_special_runtime_exit_condition() を呼んで対処する.
  
      (なお, ThreadInVMfromJava とは異なり, 
       JavaThread::handle_special_runtime_exit_condition() は, 引数を false に指定して呼び出す.
       See: ThreadInVMfromJava::~ThreadInVMfromJava())
      ---------------------------------------- -}

	    // Check for pending. async. exceptions or suspends.
	    if (_thread->has_special_runtime_exit_condition()) _thread->handle_special_runtime_exit_condition(false);
	  }
	
```


