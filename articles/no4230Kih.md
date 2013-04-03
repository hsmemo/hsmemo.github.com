---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp

### 名前(function name)
```
jint JNICALL jni_DestroyJavaVM(JavaVM *vm) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE1(hotspot_jni, DestroyJavaVM__entry, vm);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jint res = JNI_ERR;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(DestroyJavaVM, jint, (const jint&)res);
	
  {- -------------------------------------------
  (1) もし HotSpot がそもそもまだ作成されていなければ, 何もすることはないので, ここで終了.
      ---------------------------------------- -}

	  if (!vm_created) {
	    res = JNI_ERR;
	    return res;
	  }
	
  {- -------------------------------------------
  (1) カレントスレッドの名前を DestroyJavaVM に変更する？ #TODO
      ---------------------------------------- -}

	  JNIWrapper("DestroyJavaVM");
	  JNIEnv *env;
	  JavaVMAttachArgs destroyargs;
	  destroyargs.version = CurrentVersion;
	  destroyargs.name = (char *)"DestroyJavaVM";
	  destroyargs.group = NULL;
	  res = vm->AttachCurrentThread((void **)&env, (void *)&destroyargs);
	  if (res != JNI_OK) {
	    return res;
	  }
	
  {- -------------------------------------------
  (1) カレントスレッドの JavaThreadState を _thread_in_native から _thread_in_vm に変更.
      (JVM_ENTRY() マクロを使っていないので明示的に変更しないといけない)
      ---------------------------------------- -}

	  // Since this is not a JVM_ENTRY we have to set the thread state manually before entering.
	  JavaThread* thread = JavaThread::current();
	  ThreadStateTransition::transition_from_native(thread, _thread_in_vm);

  {- -------------------------------------------
  (1) Threads::destroy_vm() を呼んで HotSpot の終了処理を行う.
      ---------------------------------------- -}

	  if (Threads::destroy_vm()) {
	    // Should not change thread state, VM is gone
	    vm_created = false;
	    res = JNI_OK;
	    return res;
	  } else {
	    ThreadStateTransition::transition_and_fence(thread, _thread_in_vm, _thread_in_native);
	    res = JNI_ERR;
	    return res;
	  }
	
```


