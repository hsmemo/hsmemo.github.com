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
// rmonitor - pre-checked for validity
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::RawMonitorExit(JvmtiRawMonitor * rmonitor) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiError err = JVMTI_ERROR_NONE;
	
  {- -------------------------------------------
  (1) (以下の処理は, HotSpot の起動中(= Threads::number_of_threads() が 0)かどうかで ２通りに分かれる)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は HotSpot の起動中(= Threads::number_of_threads() が 0)の場合)
      (この段階では JavaThreads オブジェクトが作れていないので ObjectMonitor も使えない)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {

    {- -------------------------------------------
  (1.1) JvmtiPendingMonitors::exit() を呼んで 
        処理対象の JvmtiPendingMonitors を JvmtiPendingMonitors 内の pending list から外し, 
        結果をリターン.
        (pending list 中に該当する JvmtiPendingMonitors オブジェクトがなかった場合には false が返される)
        ---------------------------------------- -}

	    // No JavaThreads exist so just remove this monitor from the pending list.
	    // Bool value from exit is false if rmonitor is not in the list.
	    if (!JvmtiPendingMonitors::exit(rmonitor)) {
	      err = JVMTI_ERROR_NOT_MONITOR_OWNER;
	    }

  {- -------------------------------------------
  (1) (以下は HotSpot の起動完了後の場合)
      (この場合の処理は, カレントスレッドが JavaThread かそれ以外(VMThread or ConcurrentGCThread)かで2通りに分かれる)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    int r;
	    Thread* thread = Thread::current();
	
    {- -------------------------------------------
  (1.1) (以下は, カレントスレッドが JavaThread の場合)
        ---------------------------------------- -}

	    if (thread->is_Java_thread()) {

      {- -------------------------------------------
  (1.1.1) JvmtiRawMonitor::raw_exit() を呼んでロックを開放する.
    
          (なんだか JavaThreadState の変更処理で手こずっているようだが... #TODO)
          ---------------------------------------- -}

	      JavaThread* current_thread = (JavaThread*)thread;
	#ifdef PROPER_TRANSITIONS
	      // Not really unknown but ThreadInVMfromNative does more than we want
	      ThreadInVMfromUnknown __tiv;
	#endif /* PROPER_TRANSITIONS */
	      r = rmonitor->raw_exit(current_thread);

    {- -------------------------------------------
  (1.1) (以下は, カレントスレッドが VMThread or ConcurrentGCThread の場合)
        ---------------------------------------- -}

	    } else {

      {- -------------------------------------------
  (1.1.1) JvmtiRawMonitor::raw_exit() を呼んでロックを開放する.
          ---------------------------------------- -}

	      if (thread->is_VM_thread() || thread->is_ConcurrentGC_thread()) {
	        r = rmonitor->raw_exit(thread);
	      } else {
	        ShouldNotReachHere();
	      }
	    }
	
    {- -------------------------------------------
  (1.1) リターンする.
        
        なお, JvmtiRawMonitor::raw_exit() が失敗していた場合は, 適切なエラーをリターンする.
        * 返値が ObjectMonitor::OM_ILLEGAL_MONITOR_STATE の場合:
          JVMTI_ERROR_NOT_MONITOR_OWNER をリターン
        * 返値がその他の ObjectMonitor::OM_OK ではない値の場合: (このパスはあり得なさそうだが...)
          JVMTI_ERROR_INTERNAL をリターン
        ---------------------------------------- -}

	    if (r == ObjectMonitor::OM_ILLEGAL_MONITOR_STATE) {
	      err = JVMTI_ERROR_NOT_MONITOR_OWNER;
	    } else {
	      assert(r == ObjectMonitor::OM_OK, "raw_exit should have worked");
	      if (r != ObjectMonitor::OM_OK) {  // robustness
	        err = JVMTI_ERROR_INTERNAL;
	      }
	    }
	  }
	  return err;
	} /* end RawMonitorExit */
	
```


