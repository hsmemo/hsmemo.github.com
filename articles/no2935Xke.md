---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp

### 名前(function name)
```
  JvmtiEventMark(JavaThread *thread) :  _thread(thread),
                                         _jni_env(thread->jni_environment()) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiThreadState の例外に関する状態 (is_exception_detected(), is_exception_caught()) を待避しておく.
  
      また, 新しい JNI ローカル参照フレームも作成し, 
      Thread::set_active_handles() で処理対象のスレッドにセットしておく (See: [here](no3059hRF.html) for details).
  
      (なお, 本当は #if 0 の中のように書きたかったけど今のところ上手く動かないのでとりあえずの実装にしている, とのこと)
      ---------------------------------------- -}

	#if 0
	    _hblock = thread->active_handles();
	    _hblock->clear_thoroughly(); // so we can be safe
	#else
	    // we want to use the code above - but that needs the JNIHandle changes - later...
	    // for now, steal JNI push local frame code
	    JvmtiThreadState *state = thread->jvmti_thread_state();
	    // we are before an event.
	    // Save current jvmti thread exception state.
	    if (state != NULL) {
	      _exception_detected = state->is_exception_detected();
	      _exception_caught = state->is_exception_caught();
	    } else {
	      _exception_detected = false;
	      _exception_caught = false;
	    }
	
	    JNIHandleBlock* old_handles = thread->active_handles();
	    JNIHandleBlock* new_handles = JNIHandleBlock::allocate_block(thread);
	    assert(new_handles != NULL, "should not be NULL");
	    new_handles->set_pop_frame_link(old_handles);
	    thread->set_active_handles(new_handles);
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(thread == JavaThread::current(), "thread must be current!");

  {- -------------------------------------------
  (1) JavaFrameAnchor::make_walkable() を呼んで, 
      スタックフレームを辿れるようにしておく.
      ---------------------------------------- -}

	    thread->frame_anchor()->make_walkable(thread);
	  };
	
```


