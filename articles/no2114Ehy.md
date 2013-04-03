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
    public ThreadInfo[] getThreadInfo(long[] ids) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.ThreadImpl.getThreadInfo() を呼び出すだけ.
      ---------------------------------------- -}

	        return getThreadInfo(ids, 0);
	    }
	
```


