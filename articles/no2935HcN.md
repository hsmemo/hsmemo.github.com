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
JvmtiEnv::DestroyRawMonitor(JvmtiRawMonitor * rmonitor) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は, HotSpot の起動中(= Threads::number_of_threads() が 0)かどうかで ２通りに分かれる)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は HotSpot の起動中(= Threads::number_of_threads() が 0)の場合)
      ---------------------------------------- -}

	  if (Threads::number_of_threads() == 0) {

    {- -------------------------------------------
  (1.1) JvmtiPendingMonitors::destroy() を呼んで, JvmtiPendingMonitors 内の pending list から外しておく.
        ---------------------------------------- -}

	    // Remove this  monitor from pending raw monitors list
	    // if it has entered in onload or start phase.
	    JvmtiPendingMonitors::destroy(rmonitor);

  {- -------------------------------------------
  (1) (以下は HotSpot の起動完了後の場合)
      ---------------------------------------- -}

	  } else {

    {- -------------------------------------------
  (1.1) まず破棄対象の JvmtiRawMonitor のロックをカレントスレッドが取っているかどうかを確認し, 
        ロックを取っている場合は開放しておく.
        (しないと delete 時に assert に引っかかるため)
        (再帰的にロックを取っている可能性もあるので
         recursions() の回数だけ JvmtiRawMonitor::raw_exit() を呼び出す.
         JvmtiRawMonitor::raw_exit() が一度でも失敗したら, ここでリターン(JVMTI_ERROR_INTERNAL))
        ---------------------------------------- -}

	    Thread* thread  = Thread::current();
	    if (rmonitor->is_entered(thread)) {
	      // The caller owns this monitor which we are about to destroy.
	      // We exit the underlying synchronization object so that the
	      // "delete monitor" call below can work without an assertion
	      // failure on systems that don't like destroying synchronization
	      // objects that are locked.
	      int r;
	      intptr_t recursion = rmonitor->recursions();
	      for (intptr_t i=0; i <= recursion; i++) {
	        r = rmonitor->raw_exit(thread);
	        assert(r == ObjectMonitor::OM_OK, "raw_exit should have worked");
	        if (r != ObjectMonitor::OM_OK) {  // robustness
	          return JVMTI_ERROR_INTERNAL;
	        }
	      }
	    }

    {- -------------------------------------------
  (1.1) もし破棄対象の JvmtiRawMonitor が誰かにロックを取られていたら, 
        ここでリターン (JVMTI_ERROR_NOT_MONITOR_OWNER).
    
        (これは JVMTI 仕様では禁止されていないが, HotSpot 内の assert には引っかかるケース)
        ---------------------------------------- -}

	    if (rmonitor->owner() != NULL) {
	      // The caller is trying to destroy a monitor that is locked by
	      // someone else. While this is not forbidden by the JVMTI
	      // spec, it will cause an assertion failure on systems that don't
	      // like destroying synchronization objects that are locked.
	      // We indicate a problem with the error return (and leak the
	      // monitor's memory).
	      return JVMTI_ERROR_NOT_MONITOR_OWNER;
	    }
	  }
	
  {- -------------------------------------------
  (1) 破棄対象の JvmtiRawMonitor オブジェクトを delete する
      ---------------------------------------- -}

	  delete rmonitor;
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end DestroyRawMonitor */
	
```


