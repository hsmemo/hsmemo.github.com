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
void
JvmtiEnvBase::set_event_callbacks(const jvmtiEventCallbacks* callbacks,
                                               jint size_of_callbacks) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads::number_of_threads() == 0 || JvmtiThreadState_lock->is_locked(), "sanity check");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  size_t byte_cnt = sizeof(jvmtiEventCallbacks);
	
  {- -------------------------------------------
  (1) 最初に _event_callbacks フィールドを 0 クリアしておく.
      ---------------------------------------- -}

	  // clear in either case to be sure we got any gap between sizes
	  memset(&_event_callbacks, 0, byte_cnt);
	
  {- -------------------------------------------
  (1) memcpy() を呼んで, callbacks 引数で指定された関数ポインタを _event_callbacks フィールドに書き込む.
  
      (ただし, callbacks 引数が NULL であれば何もしない)
      (また, この JVMTI environment が既に破棄されている場合も (= JvmtiEnvBase::is_valid() が false) 何もしない 
       (See: JvmtiEnvBase::dispose())).
      ---------------------------------------- -}

	  // Now that JvmtiThreadState_lock is held, prevent a possible race condition where events
	  // are re-enabled by a call to set event callbacks where the DisposeEnvironment
	  // occurs after the boiler-plate environment check and before the lock is acquired.
	  if (callbacks != NULL && is_valid()) {
	    if (size_of_callbacks < (jint)byte_cnt) {
	      byte_cnt = size_of_callbacks;
	    }
	    memcpy(&_event_callbacks, callbacks, byte_cnt);
	  }
	}
	
```


