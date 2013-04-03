---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
void ConcurrentLocksDump::add_lock(JavaThread* thread, instanceOop o) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ThreadConcurrentLocks::add_lock() を呼んで, 
      引数で指定されたスレッド(以下の thread)用の ThreadConcurrentLocks オブジェクトに
      引数で指定された java.util.concurrent.locks.AbstractOwnableSynchronizer オブジェクト(以下の o)を登録する.
  
      (なお, スレッドに対応する ThreadConcurrentLocks オブジェクトがまだ作成されていない場合には, 
       新しい ThreadConcurrentLocks オブジェクトを作って _map フィールドや _last フィールドに登録する処理も行っている.
       作成済みかどうかは, ConcurrentLocksDump::thread_concurrent_locks() で調べられる)
      ---------------------------------------- -}

	  ThreadConcurrentLocks* tcl = thread_concurrent_locks(thread);
	  if (tcl != NULL) {
	    tcl->add_lock(o);
	    return;
	  }
	
	  // First owned lock found for this thread
	  tcl = new ThreadConcurrentLocks(thread);
	  tcl->add_lock(o);
	  if (_map == NULL) {
	    _map = tcl;
	  } else {
	    _last->set_next(tcl);
	  }
	  _last = tcl;
	}
	
```


