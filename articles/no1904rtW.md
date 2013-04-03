---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/javaCalls.cpp

### 名前(function name)
```
JavaCallWrapper::~JavaCallWrapper() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_thread == JavaThread::current(), "must still be the same thread");
	
  {- -------------------------------------------
  (1) JNI ローカル参照フレームを, 待避していた古いものに戻す.
      ---------------------------------------- -}

	  // restore previous handle block & Java frame linkage
	  JNIHandleBlock *_old_handles = _thread->active_handles();
	  _thread->set_active_handles(_handles);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  _thread->frame_anchor()->zap();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  debug_only(_thread->dec_java_call_counter());
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (_anchor.last_Java_sp() == NULL) {
	    _thread->set_base_of_stack_pointer(NULL);
	  }
	
	
  {- -------------------------------------------
  (1) JavaThreadState を _thread_in_vm に変更する.
      ---------------------------------------- -}

	  // Old thread-local info. has been restored. We are not back in the VM.
	  ThreadStateTransition::transition_from_java(_thread, _thread_in_vm);
	
  {- -------------------------------------------
  (1) JavaFrameAnchor を待避していた値に戻す.
      ---------------------------------------- -}

	  // State has been restored now make the anchor frame visible for the profiler.
	  // Do this after the transition because this allows us to put an assert
	  // the Java->vm transition which checks to see that stack is not walkable
	  // on sparc/ia64 which will catch violations of the reseting of last_Java_frame
	  // invariants (i.e. _flags always cleared on return to Java)
	
	  _thread->frame_anchor()->copy(&_anchor);
	
  {- -------------------------------------------
  (1) 使い終わった JNIHandleBlock を解放する.
      ---------------------------------------- -}

	  // Release handles after we are marked as being inside the VM again, since this
	  // operation might block
	  JNIHandleBlock::release_block(_old_handles, _thread);
	}
	
```


