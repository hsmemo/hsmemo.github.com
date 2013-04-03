---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.hpp

### 名前(function name)
```
inline void JNIHandles::destroy_local(jobject handle) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数が NULL でなければ, Thread::_active_handles 内の該当する箇所に JNIHandles::_deleted_handle の値を書き込む.
      (引数が NULL の場合は何もしない)
      ---------------------------------------- -}

	  if (handle != NULL) {
	    *((oop*)handle) = deleted_handle(); // Mark the handle as deleted, allocate will reuse it
	  }
	}
	
```


