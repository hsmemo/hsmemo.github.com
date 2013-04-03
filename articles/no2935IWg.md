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
JvmtiEnv::RawMonitorNotify(JvmtiRawMonitor * rmonitor) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int r;
	  Thread* thread = Thread::current();
	
  {- -------------------------------------------
  (1) (以下の処理は, カレントスレッドが JavaThread かそれ以外(VMThread or ConcurrentGCThread)かで2通りに分かれる)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は, カレントスレッドが JavaThread の場合)
      ---------------------------------------- -}

	  if (thread->is_Java_thread()) {

    {- -------------------------------------------
  (1.1) JvmtiRawMonitor::raw_notify() を呼んで待機しているスレッドを起床させる
        (なお, ThreadInVMfromUnknown で JavaThreadState の変更処理も行っている)
        ---------------------------------------- -}

	    JavaThread* current_thread = (JavaThread*)thread;
	    // Not really unknown but ThreadInVMfromNative does more than we want
	    ThreadInVMfromUnknown __tiv;
	    r = rmonitor->raw_notify(current_thread);

  {- -------------------------------------------
  (1) (以下は, カレントスレッドが VMThread or ConcurrentGCThread の場合)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) JvmtiRawMonitor::raw_notify() を呼んで待機しているスレッドを起床させる
        ---------------------------------------- -}

	    if (thread->is_VM_thread() || thread->is_ConcurrentGC_thread()) {
	      r = rmonitor->raw_notify(thread);
	    } else {
	      ShouldNotReachHere();
	    }
	  }
	
  {- -------------------------------------------
  (1) リターンする.
      
      なお, JvmtiRawMonitor::raw_notify() が失敗していた場合は, 適切なエラーをリターンする.
      * 返値が ObjectMonitor::OM_ILLEGAL_MONITOR_STATE の場合:
        JVMTI_ERROR_NOT_MONITOR_OWNER をリターン
      * 返値がその他の ObjectMonitor::OM_OK ではない値の場合: (このパスはあり得なさそうだが...)
        JVMTI_ERROR_INTERNAL をリターン
      ---------------------------------------- -}

	  if (r == ObjectMonitor::OM_ILLEGAL_MONITOR_STATE) {
	    return JVMTI_ERROR_NOT_MONITOR_OWNER;
	  }
	  assert(r == ObjectMonitor::OM_OK, "raw_notify should have worked");
	  if (r != ObjectMonitor::OM_OK) {  // robustness
	    return JVMTI_ERROR_INTERNAL;
	  }
	
	  return JVMTI_ERROR_NONE;
	} /* end RawMonitorNotify */
	
```


