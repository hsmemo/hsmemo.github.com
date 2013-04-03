---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
JvmtiDeferredEvent JvmtiDeferredEvent::dynamic_code_generated_event(
      const char* name, const void* code_begin, const void* code_end) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい JvmtiDeferredEvent を作成し, フィールドを初期化した後, リターンする.
  
      (なお, name 引数で渡された名前については
       消えてしまわないよう os::strdup() を呼んでメモリコピーしておく)
      ---------------------------------------- -}

	  JvmtiDeferredEvent event = JvmtiDeferredEvent(TYPE_DYNAMIC_CODE_GENERATED);
	  // Need to make a copy of the name since we don't know how long
	  // the event poster will keep it around after we enqueue the
	  // deferred event and return. strdup() failure is handled in
	  // the post() routine below.
	  event._event_data.dynamic_code_generated.name = os::strdup(name);
	  event._event_data.dynamic_code_generated.code_begin = code_begin;
	  event._event_data.dynamic_code_generated.code_end = code_end;
	  return event;
	}
	
```


