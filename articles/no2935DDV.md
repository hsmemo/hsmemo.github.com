---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp

### 名前(function name)
```
jvmtiError
JvmtiEnvBase::get_owned_monitors(JavaThread *calling_thread, JavaThread* java_thread,
                                 GrowableArray<jvmtiMonitorStackDepthInfo*> *owned_monitors_list) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;
	#ifdef ASSERT
	  uint32_t debug_bits = 0;
	#endif

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert((SafepointSynchronize::is_at_safepoint() ||
	          is_thread_fully_suspended(java_thread, false, &debug_bits)),
	         "at safepoint or target thread is suspended");
	
  {- -------------------------------------------
  (1) 処理対象のスレッド(java_thread)のスタックフレームを辿る.
      その中で JvmtiEnvBase::get_locked_objects_in_frame() を呼び出し, 取得しているモニターの情報を取得していく.
      ---------------------------------------- -}

	  if (java_thread->has_last_Java_frame()) {
	    ResourceMark rm;
	    HandleMark   hm;
	    RegisterMap  reg_map(java_thread);
	
	    int depth = 0;
	    for (javaVFrame *jvf = java_thread->last_java_vframe(&reg_map); jvf != NULL;
	         jvf = jvf->java_sender()) {
	      if (depth++ < MaxJavaStackTraceDepth) {  // check for stack too deep
	        // add locked objects for this frame into list
	        err = get_locked_objects_in_frame(calling_thread, java_thread, jvf, owned_monitors_list, depth-1);
	        if (err != JVMTI_ERROR_NONE) {
	          return err;
	        }
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) JvmtiMonitorClosure を引数として
      ObjectSynchronizer::monitors_iterate() を呼び出し, 
      jni の MonitorEnter() で取得されたモニターの情報も取得しておく.
      ---------------------------------------- -}

	  // Get off stack monitors. (e.g. acquired via jni MonitorEnter).
	  JvmtiMonitorClosure jmc(java_thread, calling_thread, owned_monitors_list, this);
	  ObjectSynchronizer::monitors_iterate(&jmc);
	  err = jmc.error();
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return err;
	}
	
```


