---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/fprofiler.cpp

### 名前(function name)
```
void ThreadProfiler::disengage() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) engaged フィールドの値を変更.
      ---------------------------------------- -}

	  engaged = false;

  {- -------------------------------------------
  (1) 呼び出された時点での時刻を記録しておく.
      ---------------------------------------- -}

	  timer.stop();
	}
	
```


