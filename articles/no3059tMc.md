---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp
### 説明(description)

```
// Thread start routine for all newly created threads
```

### 名前(function name)
```
static void *java_start(Thread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) alloca() を呼んで, スタックの位置を多少増減させておく.
      (増減させる量は, 自分自身の process id と 
       static な int 変数(以下の counter)に基づいて決めている)
    
      (なおコメントによると, これはキャッシュ上での競合を減らすための措置.
       他のスレッドのスタックとキャッシュラインがかぶっていると, お互いに invalidate しあうことになるので.
       競合は同じ HotSpot プロセス内のスレッド同士でも起こりうるし, 
       別の HotSpot プロセス内のスレッドとも起こりうる.
       特に, Hyperthreading 環境ではずらすことによる効果が大きい, とのこと.)
      ---------------------------------------- -}

	  // Try to randomize the cache line index of hot stack frames.
	  // This helps when threads of the same stack traces evict each other's
	  // cache lines. The threads can be either from the same JVM instance, or
	  // from different JVM instances. The benefit is especially true for
	  // processors with hyperthreading technology.
	  static int counter = 0;
	  int pid = os::current_process_id();
	  alloca(((pid ^ counter++) & 7) * 128);
	
  {- -------------------------------------------
  (1) ThreadLocalStorage::set_thread() を呼んで, 
      ネイティブスレッド(OS レベルのスレッド)の TLS に
      そのネイティブスレッドに対応する Thread オブジェクトをセットしておく.
  
      (ところで, この処理は JavaThread::run() の中でもやっているような... #TODO)
      ---------------------------------------- -}

	  ThreadLocalStorage::set_thread(thread);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  OSThread* osthread = thread->osthread();
	  Monitor* sync = osthread->startThread_lock();
	
  {- -------------------------------------------
  (1) _thread_safety_check() を呼んで, 新しいスレッドが作れる状況かどうかを確認しておく.
      もう作れない場合には, 以下の処理を行った後, ここでリターン.
      * カレントスレッドの OSThread::startThread_lock() に対して Monitor::notify_all() する.
        (これは生成元のスレッドとの同期処理.
         See: os::create_thread())
      * カレントスレッドの ThreadState を ZOMBIE に変更しておく.
      ---------------------------------------- -}

	  // non floating stack LinuxThreads needs extra check, see above
	  if (!_thread_safety_check(thread)) {
	    // notify parent thread
	    MutexLockerEx ml(sync, Mutex::_no_safepoint_check_flag);
	    osthread->set_state(ZOMBIE);
	    sync->notify_all();
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) os::Linux::gettid() で thread id を取得し, 
      それを OSThread::set_thread_id() で thread_id フィールドにセットしておく.
      ---------------------------------------- -}

	  // thread_id is kernel thread id (similar to Solaris LWP id)
	  osthread->set_thread_id(os::Linux::gettid());
	
  {- -------------------------------------------
  (1) UseNUMA オプションが指定されている場合は, 
      os::numa_get_group_id() で id を取得し, 
      カレントスレッドの lgrp_id フィールドにセットしておく.
      (See: [here](no28916ddy.html) for details)
      ---------------------------------------- -}

	  if (UseNUMA) {
	    int lgrp_id = os::numa_get_group_id();
	    if (lgrp_id != -1) {
	      thread->set_lgrp_id(lgrp_id);
	    }
	  }

  {- -------------------------------------------
  (1) os::Linux::hotspot_sigmask() を呼び出して, 適切なシグナルマスクを設定しておく.
      ---------------------------------------- -}

	  // initialize signal mask for this thread
	  os::Linux::hotspot_sigmask(thread);
	
  {- -------------------------------------------
  (1) os::Linux::init_thread_fpu_state() を呼び出して, 
      浮動小数点演算器に関する初期化を行っておく.
      ---------------------------------------- -}

	  // initialize floating point control register
	  os::Linux::init_thread_fpu_state();
	
  {- -------------------------------------------
  (1) (以下のブロック内で, 生成元であるスレッドとの同期処理を行う)
      ---------------------------------------- -}

	  // handshaking with parent thread
	  {
	    MutexLockerEx ml(sync, Mutex::_no_safepoint_check_flag);
	
    {- -------------------------------------------
  (1.1) まず, カレントスレッドの OSThread::startThread_lock() に対して Monitor::notify_all() を呼び出す.
        (これは生成元のスレッドとの同期処理.
         See: os::create_thread())
        ---------------------------------------- -}

	    // notify parent thread
	    osthread->set_state(INITIALIZED);
	    sync->notify_all();
	
    {- -------------------------------------------
  (1.1) 次に, カレントスレッドの OSThread::startThread_lock() に対して Monitor::wait() を呼び出して待機する.
        (これも生成元のスレッドとの同期処理.  See: os::pd_start_thread())
        この処理を, 生成元のスレッドが, カレントスレッドの ThreadState を 
        INITIALIZED に変更してくれるまで繰り返す.
        ---------------------------------------- -}

	    // wait until os::start_thread()
	    while (osthread->get_state() == INITIALIZED) {
	      sync->wait(Mutex::_no_safepoint_check_flag);
	    }
	  }
	
  {- -------------------------------------------
  (1) Thread::run() (もしくはサブクラスでオーバーライドされたメソッド) を呼び出して, 
      スレッドのメイン処理を実行する.
      ---------------------------------------- -}

	  // call one more level start routine
	  thread->run();
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return 0;
	}
	
```


