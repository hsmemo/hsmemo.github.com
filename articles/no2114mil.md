---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp
### 説明(description)

```
// Dump stack trace of threads specified in the given threads array.
// Returns StackTraceElement[][] each element is the stack trace of a thread in
// the corresponding entry in the given threads array
```

### 名前(function name)
```
Handle ThreadService::dump_stack_traces(GrowableArray<instanceHandle>* threads,
                                        int num_threads,
                                        TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(num_threads > 0, "just checking");
	
  {- -------------------------------------------
  (1) VM_ThreadDump::doit() でスタックトレースを取得する.
      ---------------------------------------- -}

	  ThreadDumpResult dump_result;
	  VM_ThreadDump op(&dump_result,
	                   threads,
	                   num_threads,
	                   -1,    /* entire stack */
	                   false, /* with locked monitors */
	                   false  /* with locked synchronizers */);
	  VMThread::execute(&op);
	
  {- -------------------------------------------
  (1) 返値は java.lang.StackTraceElement[][] としてリターンする必要があるので, 
      oopFactory::new_objArray() で新しい java.lang.StackTraceElement[][] オブジェクトを確保し
      VM_ThreadDump::doit() の結果をそちらに詰め直す.
      ---------------------------------------- -}

	  // Allocate the resulting StackTraceElement[][] object
	
	  ResourceMark rm(THREAD);
	  klassOop k = SystemDictionary::resolve_or_fail(vmSymbols::java_lang_StackTraceElement_array(), true, CHECK_NH);
	  objArrayKlassHandle ik (THREAD, k);
	  objArrayOop r = oopFactory::new_objArray(ik(), num_threads, CHECK_NH);
	  objArrayHandle result_obj(THREAD, r);
	
	  int num_snapshots = dump_result.num_snapshots();
	  assert(num_snapshots == num_threads, "Must have num_threads thread snapshots");
	  int i = 0;
	  for (ThreadSnapshot* ts = dump_result.snapshots(); ts != NULL; i++, ts = ts->next()) {
	    ThreadStackTrace* stacktrace = ts->get_stack_trace();
	    if (stacktrace == NULL) {
	      // No stack trace
	      result_obj->obj_at_put(i, NULL);
	    } else {
	      // Construct an array of java/lang/StackTraceElement object
	      Handle backtrace_h = stacktrace->allocate_fill_stack_trace_element_array(CHECK_NH);
	      result_obj->obj_at_put(i, backtrace_h());
	    }
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return result_obj;
	}
	
```


