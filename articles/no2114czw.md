---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/VMManagementImpl.java
### 説明(description)

```
    // Class Loading Subsystem
```

### 名前(function name)
```
    public int    getLoadedClassCount() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.VMManagementImpl.getTotalClassCount() でこれまでの合計ロード回数を取得し, 
      そこから sun.management.VMManagementImpl.getUnloadedClassCount() で
      取得したアンロード回数を引いた数を, リターンする.
      ---------------------------------------- -}

	        long count = getTotalClassCount() - getUnloadedClassCount();
	        return (int) count;
	    }
	
```


