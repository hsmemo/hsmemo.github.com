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
static void ensure_join(JavaThread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, この処理では Threads_lock を取得する必要は無い. なぜなら処理対象は自分自身だから)
      ---------------------------------------- -}

	  // We do not need to grap the Threads_lock, since we are operating on ourself.

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle threadObj(thread, thread->threadObj());
	  assert(threadObj.not_null(), "java thread object must exist");

  {- -------------------------------------------
  (1) この後の処理で notify_all() を呼び出す必要があるので, ここでロックを取っておく.
      ---------------------------------------- -}

	  ObjectLocker lock(threadObj, thread);

  {- -------------------------------------------
  (1) このスレッドはどのみちもう終了するので, 例外が起こっていたとしても無視する.
      そのために ThreadShadow::clear_pending_exception() を呼んで pending_exception を消しておく.
      ---------------------------------------- -}

	  // Ignore pending exception (ThreadDeath), since we are exiting anyway
	  thread->clear_pending_exception();

  {- -------------------------------------------
  (1) このスレッドはもう終了するので, 
      対応する java.lang.Thread オブジェクトの thread_status フィールドの値を TERMINATED に変更しておく.
      ---------------------------------------- -}

	  // Thread is exiting. So set thread_status field in  java.lang.Thread class to TERMINATED.
	  java_lang_Thread::set_thread_status(threadObj(), java_lang_Thread::TERMINATED);

  {- -------------------------------------------
  (1) このスレッドはもう終了するので, 
      対応する java.lang.Thread オブジェクトの eetop フィールドを NULL に戻しておく.
    
      (これにより, java.lang.Thread.isAlive() が false を返すようになる.
       See: [here](no30595cP.html) for details)
      ---------------------------------------- -}

	  // Clear the native thread instance - this makes isAlive return false and allows the join()
	  // to complete once we've done the notify_all below
	  java_lang_Thread::set_thread(threadObj(), NULL);

  {- -------------------------------------------
  (1) ObjectLocker::notify_all() を呼んで, 
      カレントスレッドに対応する java.lang.Thread オブジェクトに対して wait() で待機していたスレッドを起床させる.
  
      (これは java.lang.Thread.join() を実現するための処理.
       See: [here](no3059vOq.html) for details)
      ---------------------------------------- -}

	  lock.notify_all(thread);

  {- -------------------------------------------
  (1) このスレッドはどのみちもう終了するので, 例外が起こっていたとしても無視する.
      そのために ThreadShadow::clear_pending_exception() を呼んで pending_exception を消しておく.
      ---------------------------------------- -}

	  // Ignore pending exception (ThreadDeath), since we are exiting anyway
	  thread->clear_pending_exception();
	}
	
```


