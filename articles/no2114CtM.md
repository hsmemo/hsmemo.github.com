---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/HotspotClassLoading.java

### 名前(function name)
```
    public long getLoadedClassSize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.VMManagementImpl.getLoadedClassSize() を呼び出すだけ.
      ---------------------------------------- -}

	        return jvm.getLoadedClassSize();
	    }
	
```


