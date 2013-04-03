---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/safepoint.cpp

### 名前(function name)
```
void ThreadSafepointState::create(JavaThread *thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい ThreadSafepointState オブジェクトを作成し,  
      引数で指定されたスレッドの safepoint_state フィールドにセットする.
      (See: ThreadSafepointState)
      ---------------------------------------- -}

	  ThreadSafepointState *state = new ThreadSafepointState(thread);
	  thread->set_safepoint_state(state);
	}
	
```


