---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/space.cpp

### 名前(function name)
```
size_t TenuredSpace::allowed_dead_ratio() const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) MarkSweepDeadRatio をリターンするだけ
      ---------------------------------------- -}

	  return MarkSweepDeadRatio;
	}
	
```


