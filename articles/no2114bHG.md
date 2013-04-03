---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Returns the boolean value of a given attribute.
```

### 名前(function name)
```
JVM_LEAF(jboolean, jmm_GetBoolAttribute(JNIEnv *env, jmmBoolAttribute att))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定された種別(以下の att)に従い, 適切な関数を呼び出して, 値をリターンする.
      ---------------------------------------- -}

	  switch (att) {
	  case JMM_VERBOSE_GC:
	    return MemoryService::get_verbose();
	  case JMM_VERBOSE_CLASS:
	    return ClassLoadingService::get_verbose();
	  case JMM_THREAD_CONTENTION_MONITORING:
	    return ThreadService::is_thread_monitoring_contention();
	  case JMM_THREAD_CPU_TIME:
	    return ThreadService::is_thread_cpu_time_enabled();
	  case JMM_THREAD_ALLOCATED_MEMORY:
	    return ThreadService::is_thread_allocated_memory_enabled();
	  default:
	    assert(0, "Unrecognized attribute");
	    return false;
	  }
	JVM_END
	
```


