---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExport.cpp

### 名前(function name)
```
  ~JvmtiCompiledMethodLoadEventMark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jvmtiAddrLocationMap 情報を格納していた C ヒープ領域を開放する.
      ---------------------------------------- -}

	     FREE_C_HEAP_ARRAY(jvmtiAddrLocationMap, _map);
	  }
	
```


