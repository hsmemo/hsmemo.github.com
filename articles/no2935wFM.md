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
bool JvmtiExport::hide_single_stepping(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) thread 引数に対応する JvmtiThreadState に対して
      JvmtiThreadState::set_hide_single_stepping() を呼び出すだけ.
    
      (なお, 対応する JvmtiThreadState がまだ作られていなかったり, 
       作られていても JVMTI_EVENT_SINGLE_STEP が有効になっていない場合は
       (特にすることはないので) 何もしない)
      ---------------------------------------- -}

	  JvmtiThreadState *state = thread->jvmti_thread_state();
	  if (state != NULL && state->is_enabled(JVMTI_EVENT_SINGLE_STEP)) {
	    state->set_hide_single_stepping();
	    return true;
	  } else {
	    return false;
	  }
	}
	
```


