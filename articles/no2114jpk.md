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
    private static GarbageCollectorMXBean
        createGarbageCollector(String name, String type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい sun.management.GarbageCollectorImpl のインスタンスを作ってリターンする
  
      (なお, 第2引数の type は使っていない. これは将来の拡張用とのこと)
      ---------------------------------------- -}
	
	        // ignore type parameter which is for future extension
	        return new GarbageCollectorImpl(name);
	    }
	
```


