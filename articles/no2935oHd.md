---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectedHeap.cpp

### 名前(function name)
```
void G1CollectedHeap::wait_while_free_regions_coming() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (参考:
       G1CollectedHeap::free_regions_coming() は
       ConcurrentMarkThread の Cleanup 処理で free region が発見されると true になり, 
       ConcurrentMarkThread がそれらの free region を処理し終わると false に戻る.
       See: ConcurrentMark::cleanup(), ConcurrentMarkThread::run())
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) もし未処理の free region がなければ, 何もすることはないので, ここでリターン.
      ---------------------------------------- -}

	  // Most of the time we won't have to wait, so let's do a quick test
	  // first before we take the lock.
	  if (!free_regions_coming()) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (G1ConcRegionFreeingVerbose) {
	    gclog_or_tty->print_cr("G1ConcRegionFreeing [other] : "
	                           "waiting for free regions");
	  }
	
  {- -------------------------------------------
  (1) G1CollectedHeap::free_regions_coming() が false になるまで, ここで待機.
      (待機処理は SecondaryFreeList_lock に対して Monitor::wait() を呼ぶことで行う)
      ---------------------------------------- -}

	  {
	    MutexLockerEx x(SecondaryFreeList_lock, Mutex::_no_safepoint_check_flag);
	    while (free_regions_coming()) {
	      SecondaryFreeList_lock->wait(Mutex::_no_safepoint_check_flag);
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (G1ConcRegionFreeingVerbose) {
	    gclog_or_tty->print_cr("G1ConcRegionFreeing [other] : "
	                           "done waiting for free regions");
	  }
	}
	
```


