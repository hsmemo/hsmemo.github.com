---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp

### 名前(function name)
```
static int Stall (int its) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定された回数分(its)だけ MarsagliaXORV() を呼び出してスピンループする.
      ---------------------------------------- -}

	  static volatile jint rv = 1 ;
	  volatile int OnFrame = 0 ;
	  jint v = rv ^ UNS(OnFrame) ;
	  while (--its >= 0) {
	    v = MarsagliaXORV (v) ;
	  }
	  // Make this impossible for the compiler to optimize away,
	  // but (mostly) avoid W coherency sharing on MP systems.
	  if (v == 0x12345) rv = v ;
	  return v ;
	}
	
```


