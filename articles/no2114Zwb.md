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
    public boolean isCollectionUsageThresholdExceeded() {
```

### 本体部(body)
```
	        if (!isCollectionUsageThresholdSupported()) {
	            throw new UnsupportedOperationException(
	                "CollectionUsage threshold is not supported");
	        }
	
	        // return false if usage threshold crossing checking is disabled
	        if (collectionThreshold == 0) {
	            return false;
	        }
	
	        MemoryUsage u = getCollectionUsage0();
	        return (gcSensor.isOn() ||
	                (u != null && u.getUsed() >= collectionThreshold));
	    }
	
```


