---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::anewarray(JavaThread* thread, constantPoolOopDesc* pool, int index, jint size))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopFactory::new_objArray() でメモリの確保を行う.
        (確保した結果は, JavaThread::set_vm_result() で JavaThread 内に格納してリターン)
      ---------------------------------------- -}

	  // Note: no oopHandle for pool & klass needed since they are not used
	  //       anymore after new_objArray() and no GC can happen before.
	  //       (This may have to change if this code changes!)
	  klassOop  klass = pool->klass_at(index, CHECK);
	  objArrayOop obj = oopFactory::new_objArray(klass, size, CHECK);
	  thread->set_vm_result(obj);
	IRT_END
	
```


