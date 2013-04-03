---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/ManagementFactory.java

### 名前(function name)
```
    private static MemoryManagerMXBean createMemoryManager(String name) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい sun.management.MemoryManagerImpl のインスタンスを作ってリターンする
      ---------------------------------------- -}

	        return new MemoryManagerImpl(name);
	    }
	
```


