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
// JNI locks on java objects
// NOTE: must use heavy weight monitor to handle jni monitor enter
```

### 名前(function name)
```
void ObjectSynchronizer::jni_enter(Handle obj, TRAPS) { // possible entry from jni enter
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  // the current locking is from JNI instead of Java code
	  TEVENT (jni_enter) ;

  {- -------------------------------------------
  (1) UseBiasedLocking オプションが指定されている場合には, 
      BiasedLocking::revoke_and_rebias() を呼んで revoke しておく.
      ---------------------------------------- -}

	  if (UseBiasedLocking) {
	    BiasedLocking::revoke_and_rebias(obj, false, THREAD);
	    assert(!obj->mark()->has_bias_pattern(), "biases should be revoked by now");
	  }

  {- -------------------------------------------
  (1) ObjectSynchronizer::inflate() で ObjectMonitor オブジェクトを取得した後, 
      ObjectMonitor::enter() でロックを行う.
    
      (なお, この処理の前後でカレントスレッドの 
       current_pending_monitor_is_from_java フィールドをいじっているが, 
       このフィールドはデッドロック情報を出力する際にのみ使用される模様.
       See: DeadlockCycle::print_on())
      ---------------------------------------- -}

	  THREAD->set_current_pending_monitor_is_from_java(false);
	  ObjectSynchronizer::inflate(THREAD, obj())->enter(THREAD);
	  THREAD->set_current_pending_monitor_is_from_java(true);
	}
	
```


