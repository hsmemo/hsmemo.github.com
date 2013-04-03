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
    public void setCollectionUsageThreshold(long newThreshold) {
```

### 本体部(body)
```
	        if (!isCollectionUsageThresholdSupported()) {
	            throw new UnsupportedOperationException(
	                "CollectionUsage threshold is not supported");
	        }
	
	        Util.checkControlAccess();
	
	        MemoryUsage usage = getUsage0();
	        if (newThreshold < 0) {
	            throw new IllegalArgumentException(
	                "Invalid threshold: " + newThreshold);
	        }
	
	        if (usage.getMax() != -1 && newThreshold > usage.getMax()) {
	            throw new IllegalArgumentException(
	                "Invalid threshold: " + newThreshold +
	                     " > max (" + usage.getMax() + ").");
	        }
	
	        synchronized (this) {
	            if (!gcSensorRegistered) {
	                // pass the sensor to VM to begin monitoring
	                gcSensorRegistered = true;
	                setPoolCollectionSensor(gcSensor);
	            }
	            setCollectionThreshold0(collectionThreshold, newThreshold);
	            this.collectionThreshold = newThreshold;
	        }
	    }
	
```


