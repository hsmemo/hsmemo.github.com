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
JVM_ENTRY(void, JVM_MonitorWait(JNIEnv* env, jobject handle, jlong ms))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_MonitorWait");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle obj(THREAD, JNIHandles::resolve_non_null(handle));

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(obj->is_instance() || obj->is_array(), "JVM_MonitorWait must apply to an object");

  {- -------------------------------------------
  (1) JavaThreadInObjectWaitState で, JavaThread の状態を変更しておく.
  
      なお, JavaThreadInObjectWaitState には(プロファイル情報の記録)という役割もある.
      (See: [here](no21146np.html) for details)
      ---------------------------------------- -}

	  JavaThreadInObjectWaitState jtiows(thread, ms != 0);

  {- -------------------------------------------
  (1) (JVMTI のフック点)
      ---------------------------------------- -}

	  if (JvmtiExport::should_post_monitor_wait()) {
	    JvmtiExport::post_monitor_wait((JavaThread *)THREAD, (oop)obj(), ms);
	  }

  {- -------------------------------------------
  (1) ObjectSynchronizer::wait() を呼んで, wait 処理を行う.
      ---------------------------------------- -}

	  ObjectSynchronizer::wait(obj, ms, CHECK);
	JVM_END
	
```


