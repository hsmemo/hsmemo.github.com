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
// java_thread - unchecked
// depth - pre-checked as non-negative
// method_ptr - pre-checked for NULL
// location_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetFrameLocation(JavaThread* java_thread, jint depth, jmethodID* method_ptr, jlocation* location_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;
	  uint32_t debug_bits = 0;
	
  {- -------------------------------------------
  (1) java_thread 引数および depth 引数で指定されたスタックフレームの現在の実行地点を取得する
      取得方法は以下の2通り.
  
      * 対象のスレッドがサスペンドしている場合: 
        JvmtiEnvBase::get_frame_location() で取得.
  
      * 対象のスレッドがサスペンドしていない場合: 
        VM_GetFrameLocation で取得.
      ---------------------------------------- -}

	  if (is_thread_fully_suspended(java_thread, true, &debug_bits)) {
	    err = get_frame_location(java_thread, depth, method_ptr, location_ptr);
	  } else {
	    // JVMTI get java stack frame location at safepoint.
	    VM_GetFrameLocation op(this, java_thread, depth, method_ptr, location_ptr);
	    VMThread::execute(&op);
	    err = op.result();
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return err;
	} /* end GetFrameLocation */
	
```


