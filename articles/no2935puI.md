---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// Threads_lock NOT held, java_thread not protected by lock
// java_thread - pre-checked
// monitor_info_count_ptr - pre-checked for NULL
// monitor_info_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetOwnedMonitorStackDepthInfo(JavaThread* java_thread, jint* monitor_info_count_ptr, jvmtiMonitorStackDepthInfo** monitor_info_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;
	  JavaThread* calling_thread  = JavaThread::current();
	
  {- -------------------------------------------
  (1) 作業中に使用する配列を確保しておく.
      ---------------------------------------- -}

	  // growable array of jvmti monitors info on the C-heap
	  GrowableArray<jvmtiMonitorStackDepthInfo*> *owned_monitors_list =
	         new (ResourceObj::C_HEAP) GrowableArray<jvmtiMonitorStackDepthInfo*>(1, true);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  uint32_t debug_bits = 0;

  {- -------------------------------------------
  (1) java_thread 引数で指定されたスレッドが所有するモニター情報(およびそれらのモニターをロックしているスタックフレームの深さ)を取得する.
      取得方法は以下の2通り.
      
      * 対象のスレッドがサスペンドしている場合: 
        JvmtiEnvBase::get_owned_monitors() で取得.
  
      * 対象のスレッドがサスペンドしていない場合: 
        VM_GetOwnedMonitorInfo で取得.
      ---------------------------------------- -}

	  if (is_thread_fully_suspended(java_thread, true, &debug_bits)) {
	    err = get_owned_monitors(calling_thread, java_thread, owned_monitors_list);
	  } else {
	    // JVMTI get owned monitors info at safepoint. Do not require target thread to
	    // be suspended.
	    VM_GetOwnedMonitorInfo op(this, calling_thread, java_thread, owned_monitors_list);
	    VMThread::execute(&op);
	    err = op.result();
	  }
	
  {- -------------------------------------------
  (1) 取得処理が成功していたら, 取得した情報を
      monitor_info_count_ptr 引数および monitor_info_count_ptr 引数で指定された場所にコピーする.
      ---------------------------------------- -}

	  jint owned_monitor_count = owned_monitors_list->length();
	  if (err == JVMTI_ERROR_NONE) {
	    if ((err = allocate(owned_monitor_count * sizeof(jvmtiMonitorStackDepthInfo),
	                      (unsigned char**)monitor_info_ptr)) == JVMTI_ERROR_NONE) {
	      // copy to output array.
	      for (int i = 0; i < owned_monitor_count; i++) {
	        (*monitor_info_ptr)[i].monitor =
	          ((jvmtiMonitorStackDepthInfo*)owned_monitors_list->at(i))->monitor;
	        (*monitor_info_ptr)[i].stack_depth =
	          ((jvmtiMonitorStackDepthInfo*)owned_monitors_list->at(i))->stack_depth;
	      }
	    }
	    *monitor_info_count_ptr = owned_monitor_count;
	  }
	
  {- -------------------------------------------
  (1) 作業用の配列(及びその中身)を開放する.
      ---------------------------------------- -}

	  // clean up.
	  for (int i = 0; i < owned_monitor_count; i++) {
	    deallocate((unsigned char*)owned_monitors_list->at(i));
	  }
	  delete owned_monitors_list;
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return err;
	} /* end GetOwnedMonitorStackDepthInfo */
	
```


