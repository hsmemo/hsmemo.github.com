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
jobject JNIHandles::make_local(JNIEnv* env, oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数の oop が NULL でなければ, 
      JNIHandleBlock::allocate_handle() で Thread::_active_handles 内に格納する.
      (引数が NULL の場合は何もしない)
      ---------------------------------------- -}

	  if (obj == NULL) {
	    return NULL;                // ignore null handles
	  } else {
	    JavaThread* thread = JavaThread::thread_from_jni_environment(env);
	    assert(Universe::heap()->is_in_reserved(obj), "sanity check");
	    return thread->active_handles()->allocate_handle(obj);
	  }
	}
	
```


