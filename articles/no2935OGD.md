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
void
JvmtiEventController::change_field_watch(jvmtiEvent event_type, bool added) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiEventControllerPrivate::change_field_watch() を呼び出すだけ.
      (なお, JvmtiThreadState_lock で排他した状態で呼び出す)
      ---------------------------------------- -}

	  MutexLocker mu(JvmtiThreadState_lock);
	  JvmtiEventControllerPrivate::change_field_watch(event_type, added);
	}
	
```


