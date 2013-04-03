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
    public ThreadInfo[] dumpAllThreads(boolean lockedMonitors,
                                       boolean lockedSynchronizers) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        verifyDumpThreads(lockedMonitors, lockedSynchronizers);

  {- -------------------------------------------
  (1) sun.management.ThreadImpl.dumpThreads0() を呼び出し, その結果をリターン.
      ---------------------------------------- -}

	        return dumpThreads0(null, lockedMonitors, lockedSynchronizers);
	    }
	
```


