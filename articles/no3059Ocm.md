---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
JVM_ENTRY(void, JVM_MonitorNotify(JNIEnv* env, jobject handle))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_MonitorNotify");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle obj(THREAD, JNIHandles::resolve_non_null(handle));

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(obj->is_instance() || obj->is_array(), "JVM_MonitorNotify must apply to an object");

  {- -------------------------------------------
  (1) ObjectSynchronizer::notify() を呼んで, notify 処理を行う.
      ---------------------------------------- -}

	  ObjectSynchronizer::notify(obj, CHECK);
	JVM_END
	
```


