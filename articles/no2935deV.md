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
// monitor_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::GetCurrentContendedMonitor(JavaThread* java_thread, jobject* monitor_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;
	  uint32_t debug_bits = 0;
	  JavaThread* calling_thread  = JavaThread::current();

  {- -------------------------------------------
  (1) java_thread 引数で指定されたスレッドをブロックしているモニターの情報を取得する.
      取得方法は以下の2通り.
  
      * 対象のスレッドがサスペンドしている場合: 
        JvmtiEnvBase::get_current_contended_monitor() で取得.
  
      * 対象のスレッドがサスペンドしていない場合: 
        VM_GetCurrentContendedMonitor で取得.
      ---------------------------------------- -}

	  if (is_thread_fully_suspended(java_thread, true, &debug_bits)) {
	    err = get_current_contended_monitor(calling_thread, java_thread, monitor_ptr);
	  } else {
	    // get contended monitor information at safepoint.
	    VM_GetCurrentContendedMonitor op(this, calling_thread, java_thread, monitor_ptr);
	    VMThread::execute(&op);
	    err = op.result();
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return err;
	} /* end GetCurrentContendedMonitor */
	
```


