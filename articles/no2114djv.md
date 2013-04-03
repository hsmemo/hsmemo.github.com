---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp

### 名前(function name)
```
void Monitor::unlock() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (_owner  == Thread::current(), "invariant") ;
	  assert (_OnDeck != Thread::current()->_MutexEvent , "invariant") ;

  {- -------------------------------------------
  (1) ロックを解放するので, Monitor::set_owner() で owner をクリアしておく.
      ---------------------------------------- -}

	  set_owner (NULL) ;

  {- -------------------------------------------
  (1) _snuck フィールドが true であれば, 実際にロックを取得していたわけではないので, ここでリターン.
      (See: Monitor::lock()
      (なお, リターンする前に _snuck フィールドは false に戻しておく)
      ---------------------------------------- -}

	  if (_snuck) {
	    assert(SafepointSynchronize::is_at_safepoint() && Thread::current()->is_VM_thread(), "sneak");
	    _snuck = false;
	    return ;
	  }

  {- -------------------------------------------
  (1) Monitor::IUnlock() を呼んで, ロックの解放処理を行う.
      ---------------------------------------- -}

	  IUnlock (false) ;
	}
	
```


