---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/ClassLoadingImpl.java

### 名前(function name)
```
    public long getTotalLoadedClassCount() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.VMManagementImpl.getTotalClassCount() を呼び出すだけ.
      ---------------------------------------- -}

	        return jvm.getTotalClassCount();
	    }
	
```


