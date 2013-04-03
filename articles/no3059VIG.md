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
void Threads::remove(JavaThread* p) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の処理は Threads_lock で排他した状態で行う)
      ---------------------------------------- -}

	  // Extra scope needed for Thread_lock, so we can check
	  // that we do not remove thread without safepoint code notice
	  { MutexLocker ml(Threads_lock);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(includes(p), "p must be present");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    JavaThread* current = _thread_list;
	    JavaThread* prev    = NULL;
	
  {- -------------------------------------------
  (1) Threads::_thread_list からこのスレッドを外す.
      ---------------------------------------- -}

	    while (current != p) {
	      prev    = current;
	      current = current->next();
	    }
	
	    if (prev) {
	      prev->set_next(current->next());
	    } else {
	      _thread_list = p->next();
	    }

  {- -------------------------------------------
  (1) _number_of_threads フィールドの値を減らす.
      ---------------------------------------- -}

	    _number_of_threads--;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    oop threadObj = p->threadObj();
	    bool daemon = true;

  {- -------------------------------------------
  (1) daemon スレッドでない場合は, _number_of_non_daemon_threads フィールドの値を減らしておく.
    
      これによって, 残りの non-daemon thread の数が 1 つになったら, 
      Threads_lock に対して Monitor::notify_all() を呼んでおく.
  
      (HotSpot の終了時には, 終了処理を担当するスレッドが
       他の non-daemon thread が全て終了するまで待っているので, 
       それを起こしてあげるための処理.
       See: Threads::destroy_vm())
      ---------------------------------------- -}

	    if (threadObj == NULL || !java_lang_Thread::is_daemon(threadObj)) {
	      _number_of_non_daemon_threads--;
	      daemon = false;
	
	      // Only one thread left, do a notify on the Threads_lock so a thread waiting
	      // on destroy_vm will wake up.
	      if (number_of_non_daemon_threads() == 1)
	        Threads_lock->notify_all();
	    }

  {- -------------------------------------------
  (1) (プロファイル情報の記録) (See: ThreadService::remove_thread())
  
      以下のようなプロファイル情報が記録される.
      * PerfData 用 ("java.threads.live", "java.threads.daemon") 
        (See: UsePerfData, ThreadService)
      * JMM 用
        (See: ThreadService::get_daemon_thread_count(), ThreadService::get_live_thread_count())
      ---------------------------------------- -}

	    ThreadService::remove_thread(p, daemon);
	
  {- -------------------------------------------
  (1) JavaThread::set_terminated_value() を呼んで, 
      カレントスレッドの TerminatedTypes を _thread_terminated に変更する.
    
      (コメントによると, この処理は Safepoint コード用に必要とのこと.
       既に Threads::_thread_list から外してしまったので Safepoint コード内ではこのスレッドを関知できないが, 
       これ以降もロックを取るなどで Safepoint コード自体は何度か呼び出すので, 
       この時点で TerminatedTypes を変更しておかないとおかしなことになる.)
      ---------------------------------------- -}

	    // Make sure that safepoint code disregard this thread. This is needed since
	    // the thread might mess around with locks after this point. This can cause it
	    // to do callbacks into the safepoint code. However, the safepoint code is not aware
	    // of this thread since it is removed from the queue.
	    p->set_terminated_value();

  {- -------------------------------------------
  (1) (ここまでが Threads_lock で排他した処理)
      ---------------------------------------- -}

	  } // unlock Threads_lock
	
  {- -------------------------------------------
  (1) (トレース出力)
  
      (なお, Events::log() はロックを使うので Threads_lock を手放した後で呼び出している, とのこと)
      ---------------------------------------- -}

	  // Since Events::log uses a lock, we grab it outside the Threads_lock
	  Events::log("Thread exited: " INTPTR_FORMAT, p);
	}
	
```


