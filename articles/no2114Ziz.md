---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/MemoryPoolImpl.java

### 名前(function name)
```
        void triggerAction(MemoryUsage usage) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MemoryImpl.createNotification() を呼び出して, リスナーに通知を行う.
      ---------------------------------------- -}

	            MemoryImpl.createNotification(MEMORY_COLLECTION_THRESHOLD_EXCEEDED,
	                                          pool.getName(),
	                                          usage,
	                                          gcSensor.getCount());
	        }
	
```


