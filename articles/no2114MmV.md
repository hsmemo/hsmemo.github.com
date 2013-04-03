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
    public long getUsageThresholdCount() {
```

### 本体部(body)
```
	        if (!isUsageThresholdSupported()) {
	            throw new UnsupportedOperationException(
	                "Usage threshold is not supported");
	        }
	
	        return usageSensor.getCount();
	    }
	
```


