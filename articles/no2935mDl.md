---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp

### 名前(function name)
```
void JvmtiEventControllerPrivate::enter_interp_only_mode(JvmtiThreadState *state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [%s] # Entering interpreter only mode",
	            JvmtiTrace::safe_get_thread_name(state->get_thread())));
	
  {- -------------------------------------------
  (1) VM_EnterInterpOnlyMode を呼び出すだけ.
      ---------------------------------------- -}

	  VM_EnterInterpOnlyMode op(state);
	  VMThread::execute(&op);
	}
	
```


