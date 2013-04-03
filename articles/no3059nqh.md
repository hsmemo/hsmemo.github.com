---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp
### 説明(description)

```
// NOTE: must use heavy weight monitor to handle jni monitor exit
```

### 名前(function name)
```
void ObjectSynchronizer::jni_exit(oop obj, Thread* THREAD) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TEVENT (jni_exit) ;

  {- -------------------------------------------
  (1) UseBiasedLocking オプションが指定されている場合には, 
      BiasedLocking::revoke_and_rebias() を呼んで revoke しておく.
      ---------------------------------------- -}

	  if (UseBiasedLocking) {
	    BiasedLocking::revoke_and_rebias(obj, false, THREAD);
	  }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
	
  {- -------------------------------------------
  (1) ObjectSynchronizer::inflate() で ObjectMonitor オブジェクトを取得.
      ---------------------------------------- -}

	  ObjectMonitor* monitor = ObjectSynchronizer::inflate(THREAD, obj);

  {- -------------------------------------------
  (1) ObjectMonitor::check() で, カレントスレッドがロックを確保していることを確認しておく.
      ロックを確保していれば, ObjectMonitor::exit() でロックの開放処理を行う.
      (ロックを確保していなかった場合には, ObjectMonitor::check() が IllegalMonitorStateException を出す)
    
      (なおコメントによると, ObjectMonitor::check() の呼び出し時に CHECK マクロを使ってはいけないとのこと.
       ObjectMonitor::check() が true を返す場合には, 例外が起きていても exit 処理は行わなければいけない.)
      ---------------------------------------- -}

	  // If this thread has locked the object, exit the monitor.  Note:  can't use
	  // monitor->check(CHECK); must exit even if an exception is pending.
	  if (monitor->check(THREAD)) {
	     monitor->exit(THREAD);
	  }
	}
	
```


