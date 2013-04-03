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
    public Map<String, Long> getInternalThreadCpuTimes() {
```

### 本体部(body)
```
	        int count = getInternalThreadCount();
	        if (count == 0) {
	            return java.util.Collections.emptyMap();
	        }
	        String[] names = new String[count];
	        long[] times = new long[count];
	        int numThreads = getInternalThreadTimes0(names, times);
	        Map<String, Long> result = new HashMap<String, Long>(numThreads);
	        for (int i = 0; i < numThreads; i++) {
	            result.put(names[i], new Long(times[i]));
	        }
	        return result;
	    }
	
```


