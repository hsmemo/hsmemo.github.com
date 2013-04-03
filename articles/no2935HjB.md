---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRawMonitor.hpp
### 説明(description)

```
  // Return false if monitor is not found in the list.
```

### 名前(function name)
```
  static bool exit(JvmtiRawMonitor *monitor) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) monitor 引数で指定された JvmtiRawMonitor オブジェクトが monitors() 中に含まれていれば, 
      それを取り除き true をリターンする.
    
      含まれていなければ, 単に false をリターンするだけ.
      ---------------------------------------- -}

	    if (monitors()->contains(monitor)) {
	      monitors()->remove(monitor);
	      return true;
	    } else {
	      return false;
	    }
	  }
	
```


