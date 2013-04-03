---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp

### 名前(function name)
```
void ObjectSynchronizer::notify(Handle obj, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし UseBiasedLocking オプションが指定されていた場合には, BiasedLocking::revoke_and_rebias() で biased を解除しておく.
      ---------------------------------------- -}

	 if (UseBiasedLocking) {
	    BiasedLocking::revoke_and_rebias(obj, false, THREAD);
	    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
	  }
	
  {- -------------------------------------------
  (1) もし, オブジェクトが stack-locked 状態で (= wait() で待っているスレッドはいない状態で), 
      きちんと自分がそのロックを持っている場合には, 何もせずに終了.
      ---------------------------------------- -}

	  markOop mark = obj->mark();
	  if (mark->has_locker() && THREAD->is_lock_owned((address)mark->locker())) {
	    return;
	  }

  {- -------------------------------------------
  (1) ObjectSynchronizer::inflate() で ObjectMonitor を取得し, ObjectMonitor::notify() を呼び出す
      ---------------------------------------- -}

	  ObjectSynchronizer::inflate(THREAD, obj())->notify(THREAD);
	}
	
```


