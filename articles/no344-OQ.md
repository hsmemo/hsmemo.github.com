---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/threadLocalAllocBuffer.cpp

### 名前(function name)
```
void ThreadLocalAllocBuffer::startup_initialization() {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) TLAB が使用する大域変数を初期化する
      ---------------------------------------- -}

	  // Assuming each thread's active tlab is, on average,
	  // 1/2 full at a GC
	  _target_refills = 100 / (2 * TLABWasteTargetPercent);
	  _target_refills = MAX2(_target_refills, (unsigned)1U);
	
	  _global_stats = new GlobalTLABStats();
	
  {- -------------------------------------------
  (1) メインスレッドの TLAB を初期化する
      (メインスレッドについてだけは, TLAB の初期化よりスレッドの生成の方が速いため, ここで初期化してやる必要がある.
       See: TLAB)
      ---------------------------------------- -}

	  // During jvm startup, the main (primordial) thread is initialized
	  // before the heap is initialized.  So reinitialize it now.
	  guarantee(Thread::current()->is_Java_thread(), "tlab initialization thread not Java thread");
	  Thread::current()->tlab().initialize();
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintTLAB && Verbose) {
	    gclog_or_tty->print("TLAB min: " SIZE_FORMAT " initial: " SIZE_FORMAT " max: " SIZE_FORMAT "\n",
	                        min_size(), Thread::current()->tlab().initial_desired_size(), max_size());
	  }
	}
	
```


