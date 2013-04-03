---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/invocationCounter.hpp

### 名前(function name)
```
inline void InvocationCounter::decay() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) この InvocationCounter の値を半分にする.
  
      (ただし, 値が 1 の場合は (半減して 0 になると実行されていないメソッドと勘違いされてまずいので) 1 のままにしておく)
      ---------------------------------------- -}

	  int c = count();
	  int new_count = c >> 1;
	  // prevent from going to zero, to distinguish from never-executed methods
	  if (c > 0 && new_count == 0) new_count = 1;
	  set(state(), new_count);
	}
	
```


