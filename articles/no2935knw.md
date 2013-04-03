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
void JvmtiExport::expose_single_stepping(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) thread 引数に対応する JvmtiThreadState に対して
      JvmtiThreadState::clear_hide_single_stepping() を呼び出すだけ.
      ---------------------------------------- -}

	  JvmtiThreadState *state = thread->jvmti_thread_state();
	  if (state != NULL) {
	    state->clear_hide_single_stepping();
	  }
	}
	
```


