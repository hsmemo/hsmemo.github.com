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
JVM_ENTRY(jobject, JVM_CurrentThread(JNIEnv* env, jclass threadClass))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper)
      ---------------------------------------- -}

	  JVMWrapper("JVM_CurrentThread");

  {- -------------------------------------------
  (1) JVM_ENTRY マクロ内で, カレントスレッドを 
      JavaThread::thread_from_jni_environment() で取得して 
      thread という変数に束縛済みなので, 
      それを JNI Handle 化して返すだけ.
      ---------------------------------------- -}

	  oop jthread = thread->threadObj();
	  assert (thread != NULL, "no current thread!");
	  return JNIHandles::make_local(env, jthread);
	JVM_END
	
```


