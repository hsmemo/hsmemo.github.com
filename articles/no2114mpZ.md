---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void Thread::interrupt(Thread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  trace("interrupt", thread);

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  debug_only(check_for_dangling_thread_pointer(thread);)

  {- -------------------------------------------
  (1) os::interrupt() を呼んで, 割り込み処理を行う.
      ---------------------------------------- -}

	  os::interrupt(thread);
	}
	
```


