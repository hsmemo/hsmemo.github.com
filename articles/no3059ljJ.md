---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/vm_operations.cpp

### 名前(function name)
```
int VM_Exit::wait_for_threads_in_native_to_block() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // VM exits at safepoint. This function must be called at the final safepoint
	  // to wait for threads in _thread_in_native state to be quiescent.
	  assert(SafepointSynchronize::is_at_safepoint(), "must be at safepoint already");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Thread * thr_cur = ThreadLocalStorage::get_thread_slow();
	  Monitor timer(Mutex::leaf, "VM_Exit timer", true);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (max_wait_user_thread 及び max_wait (= max_wait_compiler_thread) が最大待機時間を示す.
       なお, 待機処理を行うループ内では 10 msec 単位でスリープするので, 実際の待機時間とは 10 倍のずれがある.
       つまり, それぞれ 300 msec, 及び 10 sec を表す.)
  
      (コメントによると, 
       user thread (JavaThread) の場合は, HotSpot 内のデータにアクセスするには native=>Java/VM transition を行う必要があり, 
       そこで停止させられるので, 待機時間は比較的短くていい.
       一方 CompilerThread は, native=>Java/VM transition 無しで HotSpot 内のデータにアクセスできるため, 
       長めの待機時間を取っておく必要がある.
       なお, 理屈の上では JavaThread については待たなくてもいいが, 
       HotSpot が終了する際にはスレッドが全部停止している方が望ましいので, 待機している.)
      ---------------------------------------- -}

	  // Compiler threads need longer wait because they can access VM data directly
	  // while in native. If they are active and some structures being used are
	  // deleted by the shutdown sequence, they will crash. On the other hand, user
	  // threads must go through native=>Java/VM transitions first to access VM
	  // data, and they will be stopped during state transition. In theory, we
	  // don't have to wait for user threads to be quiescent, but it's always
	  // better to terminate VM when current thread is the only active thread, so
	  // wait for user threads too. Numbers are in 10 milliseconds.
	  int max_wait_user_thread = 30;                  // at least 300 milliseconds
	  int max_wait_compiler_thread = 1000;            // at least 10 seconds
	
	  int max_wait = max_wait_compiler_thread;
	
  {- -------------------------------------------
  (1) 以下の while ループ内で, native method を実行しているスレッドの終了を待ち受ける.
  
      ループ内では, まず JavaThread の一覧を辿り, それらの内で native method を実行中のスレッドの個数を数える. 
      (また, native method を実行中のスレッドの内で CompilerThread である数も同時に数える)
      その後, 数えた結果をもとにループを終了するかどうかを判断する.
    
      以下のどれかの条件が成り立てば, ループは終了.
      * native method を実行しているスレッドがいない場合 (以下の num_active が 0 の場合)
      * ループ回数(以下の attempts)が max_wait 回数を超えた場合
      * native method を実行しているのが user thread (JavaThread) だけであり, かつ
        ループ回数(以下の attempts)が max_wait_user_thread 回数を超えた場合
      ---------------------------------------- -}

	  int attempts = 0;
	  while (true) {
	    int num_active = 0;
	    int num_active_compiler_thread = 0;
	
	    for(JavaThread *thr = Threads::first(); thr != NULL; thr = thr->next()) {
	      if (thr!=thr_cur && thr->thread_state() == _thread_in_native) {
	        num_active++;
	        if (thr->is_Compiler_thread()) {
	          num_active_compiler_thread++;
	        }
	      }
	    }
	
	    if (num_active == 0) {
	       return 0;
	    } else if (attempts > max_wait) {
	       return num_active;
	    } else if (num_active_compiler_thread == 0 && attempts > max_wait_user_thread) {
	       return num_active;
	    }
	
	    attempts++;
	
	    MutexLockerEx ml(&timer, Mutex::_no_safepoint_check_flag);
	    timer.wait(Mutex::_no_safepoint_check_flag, 10);
	  }
	}
	
```


