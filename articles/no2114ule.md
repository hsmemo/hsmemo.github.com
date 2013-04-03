---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/HotspotThread.java

### 名前(function name)
```
    public java.util.List<Counter> getInternalThreadingCounters() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.VMManagementImpl.getInternalCounters() を呼び出すだけ
      ---------------------------------------- -}

	        return jvm.getInternalCounters(THREADS_COUNTER_NAME_PATTERN);
	    }
	
```


