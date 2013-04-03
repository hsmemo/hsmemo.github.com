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
// -----------------------------------------------------------------------------
//  Wait/Notify/NotifyAll
// NOTE: must use heavy weight monitor to handle wait()
```

### 名前(function name)
```
void ObjectSynchronizer::wait(Handle obj, jlong millis, TRAPS) {
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
  (1) もし指定されたタイムアウト時間がマイナスだったら java_lang_IllegalArgumentException
      ---------------------------------------- -}

	  if (millis < 0) {
	    TEVENT (wait - throw IAX) ;
	    THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(), "timeout value is negative");
	  }

  {- -------------------------------------------
  (1) ObjectSynchronizer::inflate() で ObjectMonitor を取得し, ObjectMonitor::wait() を呼び出す
      ---------------------------------------- -}

	  ObjectMonitor* monitor = ObjectSynchronizer::inflate(THREAD, obj());
	  DTRACE_MONITOR_WAIT_PROBE(monitor, obj(), THREAD, millis);
	  monitor->wait(millis, true, THREAD);
	
  {- -------------------------------------------
  (1) (DTrace のフック点)
      (現在は DTrace のバグのため使用不可)
      ---------------------------------------- -}

	  /* This dummy call is in place to get around dtrace bug 6254741.  Once
	     that's fixed we can uncomment the following line and remove the call */
	  // DTRACE_MONITOR_PROBE(waited, monitor, obj(), THREAD);
	  dtrace_waited_probe(monitor, obj, THREAD);
	}
	
```


