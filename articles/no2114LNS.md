---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// Enqueue a VM_Operation to do the job for us - sometime later
```

### 名前(function name)
```
void Thread::send_async_exception(oop java_thread, oop java_throwable) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) VM_ThreadStop を使って, 処理対象のスレッド内で指定された例外を発生させる.
      ---------------------------------------- -}

	  VM_ThreadStop* vm_stop = new VM_ThreadStop(java_thread, java_throwable);
	  VMThread::execute(vm_stop);
	}
	
```


