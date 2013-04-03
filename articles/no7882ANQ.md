---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/concurrentGCThread.cpp

### 名前(function name)
```
static void _sltLoop(JavaThread* thread, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) thread 引数で渡された SurrogateLockerThread オブジェクトに対して, 
      SurrogateLockerThread::loop() を呼び出すだけ.
      ---------------------------------------- -}

	  SurrogateLockerThread* slt = (SurrogateLockerThread*)thread;
	  slt->loop();
	}
	
```


