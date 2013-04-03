---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void Thread::start(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  trace("start", thread);

  {- -------------------------------------------
  (1) os::start_thread() で処理対象のスレッドを開始させる.
      (なお, 処理対象が JavaThread の場合には, 
       java_lang_Thread::set_thread_status() で
       スレッドの状態を java_lang_Thread::RUNNABLE にする処理も行う)
  
      (ただし, デバッグ用のオプションである DisableStartThread が指定されている場合には, 何もしない)
      ---------------------------------------- -}

	  // Start is different from resume in that its safety is guaranteed by context or
	  // being called from a Java method synchronized on the Thread object.
	  if (!DisableStartThread) {
	    if (thread->is_Java_thread()) {
	      // Initialize the thread state to RUNNABLE before starting this thread.
	      // Can not set it after the thread started because we do not know the
	      // exact thread state at that time. It could be in MONITOR_WAIT or
	      // in SLEEPING or some other state.
	      java_lang_Thread::set_thread_status(((JavaThread*)thread)->threadObj(),
	                                          java_lang_Thread::RUNNABLE);
	    }
	    os::start_thread(thread);
	  }
	}
	
```


