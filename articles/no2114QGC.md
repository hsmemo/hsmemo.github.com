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
void VM_ThreadDump::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm;
	
  {- -------------------------------------------
  (1) 現在ロックされているシンクロナイザ一覧も取得するように指定されていれば
      (= _with_locked_synchronizers コンストラクタ引数が true であれば), 
      ConcurrentLocksDump::dump_at_safepoint() で取得しておく.
    
      (なおシンクロナイザとは, java.util.concurrent.locks.AbstractOwnableSynchronizer やそのサブクラスのインスタンスのこと.
       See: java.lang.management.ThreadMXBean.getThreadInfo(), java.lang.management.ThreadMXBean.dumpAllThreads())
      ---------------------------------------- -}

	  ConcurrentLocksDump concurrent_locks(true);
	  if (_with_locked_synchronizers) {
	    concurrent_locks.dump_at_safepoint();
	  }
	
  {- -------------------------------------------
  (1) 処理対象の JavaThread を全て辿り, 
      VM_ThreadDump::snapshot_thread() で ThreadSnapshot オブジェクトを作成していく.
      (作った ThreadSnapshot オブジェクトは
       ThreadDumpResult::add_thread_snapshot() で _result フィールド内に格納していく)
  
      なお正確には, コンストラクタ引数で処理対象のスレッドが限定されているかどうかで２通りに分岐している.
      * 処理対象のスレッドが限定されていない場合 (_num_threads が 0 の場合): 
        全てのスレッドを辿る
      * 〃 限定されている場合 (_num_threads が 0 ではない場合): 
        指定のスレッドだけを辿る.
  
      また, 以下のようなスレッドについては特別に扱う.
      * 処理対象のスレッドが限定されていない場合 (_num_threads が 0 の場合): 
        * 既に終了しているスレッド, または HotSpot の内部的なスレッド:
          結果には含めない (単に無視する)
      * 〃 限定されている場合 (_num_threads が 0 ではない場合): 
        * 指定されたスレッドが, 存在しないスレッドだった場合: 
          結果の対応する箇所には, 空の ThreadSnapshot オブジェクト (ダミーの ThreadSnapshot オブジェクト) を入れておく
        * 指定されたスレッドが既に終了している場合, または HotSpot の内部的なスレッドの場合:
          結果の対応する箇所には, 空の ThreadSnapshot オブジェクト (ダミーの ThreadSnapshot オブジェクト) を入れておく
      ---------------------------------------- -}

	  if (_num_threads == 0) {
	    // Snapshot all live threads
	    for (JavaThread* jt = Threads::first(); jt != NULL; jt = jt->next()) {
	      if (jt->is_exiting() ||
	          jt->is_hidden_from_external_view())  {
	        // skip terminating threads and hidden threads
	        continue;
	      }
	      ThreadConcurrentLocks* tcl = NULL;
	      if (_with_locked_synchronizers) {
	        tcl = concurrent_locks.thread_concurrent_locks(jt);
	      }
	      ThreadSnapshot* ts = snapshot_thread(jt, tcl);
	      _result->add_thread_snapshot(ts);
	    }
	  } else {
	    // Snapshot threads in the given _threads array
	    // A dummy snapshot is created if a thread doesn't exist
	    for (int i = 0; i < _num_threads; i++) {
	      instanceHandle th = _threads->at(i);
	      if (th() == NULL) {
	        // skip if the thread doesn't exist
	        // Add a dummy snapshot
	        _result->add_thread_snapshot(new ThreadSnapshot());
	        continue;
	      }
	
	      // Dump thread stack only if the thread is alive and not exiting
	      // and not VM internal thread.
	      JavaThread* jt = java_lang_Thread::thread(th());
	      if (jt == NULL || /* thread not alive */
	          jt->is_exiting() ||
	          jt->is_hidden_from_external_view())  {
	        // add a NULL snapshot if skipped
	        _result->add_thread_snapshot(new ThreadSnapshot());
	        continue;
	      }
	      ThreadConcurrentLocks* tcl = NULL;
	      if (_with_locked_synchronizers) {
	        tcl = concurrent_locks.thread_concurrent_locks(jt);
	      }
	      ThreadSnapshot* ts = snapshot_thread(jt, tcl);
	      _result->add_thread_snapshot(ts);
	    }
	  }
	}
	
```


