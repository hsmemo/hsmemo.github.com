---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp

### 名前(function name)
```
static int Adjust (volatile int * adr, int dx) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Atomic::cmpxchg() で値をインクリメントする (成功するまで繰り返す).
      ---------------------------------------- -}

	  int v ;
	  for (v = *adr ; Atomic::cmpxchg (v + dx, adr, v) != v; v = *adr) ;
	  return v ;
	}
	
```


