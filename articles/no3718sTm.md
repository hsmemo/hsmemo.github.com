---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jniHandles.cpp

### 名前(function name)
```
void JNIHandles::destroy_global(jobject handle) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数が NULL でなければ, JNIHandles::_global_handles 内の該当する箇所に JNIHandles::_deleted_handle の値を書き込む.
      (引数が NULL の場合は何もしない)
      ---------------------------------------- -}

	  if (handle != NULL) {
	    assert(is_global_handle(handle), "Invalid delete of global JNI handle");
	    *((oop*)handle) = deleted_handle(); // Mark the handle as deleted, allocate will reuse it
	  }
	}
	
```


