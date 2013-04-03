---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/MemoryImpl.java

### 名前(function name)
```
    static synchronized MemoryManagerMXBean[] getMemoryManagers() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.MemoryImpl.getMemoryManagers0() の結果を返すだけ.
      (既にキャッシュしてあった値があればそれを返すだけ)
      ---------------------------------------- -}

	        if (mgrs == null) {
	            mgrs = getMemoryManagers0();
	        }
	        return mgrs;
	    }
	
```


