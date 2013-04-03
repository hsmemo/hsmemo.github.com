---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/threadCritical_solaris.cpp

### 名前(function name)
```
ThreadCritical::~ThreadCritical() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ThreadCritical::initialize() が呼ばれるまでは 
       (= initialize が false であれば) 何もしない)
      ---------------------------------------- -}

	  if (initialized) {

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(global_mut_owner == thr_self(), "must have correct owner");
	    assert(global_mut_count > 0, "must have correct count");

  {- -------------------------------------------
  (1) ロックの再帰確保数をデクリメントする.
      もし確保数が 0 になったら, os::Solaris::mutex_unlock() でロックを解放する.
      ---------------------------------------- -}

	    --global_mut_count;
	    if (global_mut_count == 0) {
	      global_mut_owner = -1;
	      if (os::Solaris::mutex_unlock(&global_mut))
	        fatal(err_msg("ThreadCritical::~ThreadCritical: mutex_unlock failed "
	                      "(%s)", strerror(errno)));
	    }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  } else {
	    assert (Threads::number_of_threads() == 0, "valid only during initialization");
	  }
	}
	
```


