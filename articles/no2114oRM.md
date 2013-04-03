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
// Returns the long value of a given attribute.
```

### 名前(function name)
```
JVM_ENTRY(jlong, jmm_GetLongAttribute(JNIEnv *env, jobject obj, jmmLongAttribute att))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数で GCMemoryManager オブジェクトが渡されてきていれば, get_gc_attribute() を呼び出す.
      そうでなければ get_long_attribute() を呼び出す.
      ---------------------------------------- -}

	  if (obj == NULL) {
	    return get_long_attribute(att);
	  } else {
	    GCMemoryManager* mgr = get_gc_memory_manager_from_jobject(obj, CHECK_(0L));
	    if (mgr != NULL) {
	      return get_gc_attribute(mgr, att);
	    }
	  }
	  return -1;
	JVM_END
	
```


