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
  ~JvmtiEventMark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 処理対象のスレッドの JNI ローカル参照フレームを 1つポップする.
      (今まで使っていた JNIHandleBlock は JNIHandleBlock::release_block() でフリーリストにつないでおく)
  
      (なお, 本当は #if 0 の中のように書きたかったけど今のところ上手く動かないのでとりあえずの実装にしている, とのこと)
      ---------------------------------------- -}

	#if 0
	    _hblock->clear(); // for consistency with future correct behavior
	#else
	    // we want to use the code above - but that needs the JNIHandle changes - later...
	    // for now, steal JNI pop local frame code
	    JNIHandleBlock* old_handles = _thread->active_handles();
	    JNIHandleBlock* new_handles = old_handles->pop_frame_link();
	    assert(new_handles != NULL, "should not be NULL");
	    _thread->set_active_handles(new_handles);
	    // Note that we set the pop_frame_link to NULL explicitly, otherwise
	    // the release_block call will release the blocks.
	    old_handles->set_pop_frame_link(NULL);
	    JNIHandleBlock::release_block(old_handles, _thread); // may block
	#endif
	
  {- -------------------------------------------
  (1) JvmtiThreadState の例外に関する状態 (is_exception_detected(), is_exception_caught()) を元に戻す.
      ---------------------------------------- -}

	    JvmtiThreadState* state = _thread->jvmti_thread_state();
	    // we are continuing after an event.
	    if (state != NULL) {
	      // Restore the jvmti thread exception state.
	      if (_exception_detected) {
	        state->set_exception_detected();
	      }
	      if (_exception_caught) {
	        state->set_exception_caught();
	      }
	    }
	  }
	
```


