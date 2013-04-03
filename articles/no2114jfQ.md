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
JavaThread::JavaThread(ThreadFunction entry_point, size_t stack_sz) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化 (See: Thread::Thread())
      ---------------------------------------- -}

	  Thread()

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	#ifndef SERIALGC
	  , _satb_mark_queue(&_satb_mark_queue_set),
	  _dirty_card_queue(&_dirty_card_queue_set)
	#endif // !SERIALGC
	{

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceThreadEvents) {
	    tty->print_cr("creating thread %p", this);
	  }

  {- -------------------------------------------
  (1) JavaThread::initialize() を呼んで, 
      JavaThread オブジェクト内の種々のフィールドを初期化する.
      ---------------------------------------- -}

	  initialize();

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _is_attaching = false;
	  set_entry_point(entry_point);

  {- -------------------------------------------
  (1) os::create_thread() を呼んで, 新しいスレッドを生成する.
      ---------------------------------------- -}

	  // Create the native thread itself.
	  // %note runtime_23
	  os::ThreadType thr_type = os::java_thread;
	  thr_type = entry_point == &compiler_thread_entry ? os::compiler_thread :
	                                                     os::java_thread;
	  os::create_thread(this, thr_type, stack_sz);
	
  {- -------------------------------------------
  (1) (なお, メモリが足りなかった場合には, ここの段階では _osthread が NULL になっている.
       OutOfMemoryError を投げるべきだが, 呼び出し元がロックを握っているかもしれないので, ここでは投げられない.
       というのも, 例外を投げるには例外オブジェクトを作って初期化する作業が必要で, 
       初期化作業には JavaCall をつかって Java のコードに移る(= VM 外に出る)必要があって, 
       そのためにはロックを全部解放しておかないといけないので.)
    
      (なお, この段階ではまだスレッドは suspend された状態になっているので, 
       生成した人がスレッドを明示的に開始させないといけない.
       また, この後で Threads:add() を呼んで明示的にスレッドを登録しないといけないことにも注意.
       なぜ Threads:add() してないかというと, 完全に初期化が終わってから登録しないといけないから (JVM_Start も参照))
    
      <= と書いてあるが, JVM_Start() ってなんだ? JVM_StartThread() のことか??  #TODO
      ---------------------------------------- -}

	  // The _osthread may be NULL here because we ran out of memory (too many threads active).
	  // We need to throw and OutOfMemoryError - however we cannot do this here because the caller
	  // may hold a lock and all locks must be unlocked before throwing the exception (throwing
	  // the exception consists of creating the exception object & initializing it, initialization
	  // will leave the VM via a JavaCall and then all locks must be unlocked).
	  //
	  // The thread is still suspended when we reach here. Thread must be explicit started
	  // by creator! Furthermore, the thread must also explicitly be added to the Threads list
	  // by calling Threads:add. The reason why this is not done here, is because the thread
	  // object must be fully initialized (take a look at JVM_Start)
	}
	
```


