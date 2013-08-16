---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
void Parker::unpark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 対象のイベントオブジェクトに対して, SetEvent() でシグナルを送るだけ.
      ---------------------------------------- -}

	  guarantee (_ParkEvent != NULL, "invariant") ;
	  SetEvent(_ParkEvent);
	}
	
```


