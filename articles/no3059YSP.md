---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp
### 説明(description)

```
// NamedThread --  non-JavaThread subclasses with multiple
// uniquely named instances should derive from this.
```

### 名前(function name)
```
NamedThread::NamedThread() : Thread() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  _name = NULL;
	  _processed_thread = NULL;
	}
	
```


