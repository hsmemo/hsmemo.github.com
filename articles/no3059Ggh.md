---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/osThread.cpp

### 名前(function name)
```
OSThread::OSThread(OSThreadStartFunc start_proc, void* start_parm) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) OSThread::pd_initialize() を呼んで, 
      プラットフォーム固有の初期化処理を(もしそんなものがあれば)実行しておく
      ---------------------------------------- -}

	  pd_initialize();

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  set_start_proc(start_proc);
	  set_start_parm(start_parm);
	  set_interrupted(false);
	}
	
```


