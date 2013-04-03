---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/ThreadImpl.java

### 名前(function name)
```
    public long getThreadAllocatedBytes(long id) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.ThreadImpl.getThreadAllocatedBytes(long[] ids) を呼び出すだけ.
      ---------------------------------------- -}

	        long[] ids = new long[1];
	        ids[0] = id;
	        final long[] sizes = getThreadAllocatedBytes(ids);
	        return sizes[0];
	    }
	
```


