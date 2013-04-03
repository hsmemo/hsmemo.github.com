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
    public long getUnloadedClassCount() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.VMManagementImpl.getUnloadedClassCount() を呼び出すだけ.
      ---------------------------------------- -}

	        return jvm.getUnloadedClassCount();
	    }
	
```


