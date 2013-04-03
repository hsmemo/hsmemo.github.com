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
    public ThreadInfo getThreadInfo(long id) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.ThreadImpl.getThreadInfo() を呼び出すだけ.
      ---------------------------------------- -}

	        long[] ids = new long[1];
	        ids[0] = id;
	        final ThreadInfo[] infos = getThreadInfo(ids, 0);
	        return infos[0];
	    }
	
```


