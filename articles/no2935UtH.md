---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRawMonitor.cpp
### 説明(description)

```
// Used mainly for JVMTI raw monitor implementation
// Also used for JvmtiRawMonitor::wait().
```

### 名前(function name)
```
int JvmtiRawMonitor::raw_exit(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TEVENT (raw_exit) ;

  {- -------------------------------------------
  (1) カレントスレッドがロックを持っていなければ, ここでリターン (OM_ILLEGAL_MONITOR_STATE).
      ---------------------------------------- -}

	  if (THREAD != _owner) {
	    return OM_ILLEGAL_MONITOR_STATE;
	  }

  {- -------------------------------------------
  (1) 再帰的にロックを取られている場合(= _recursions が 1 以上の場合)は, 
      _recursions を 1 デクリメントするだけでいい.
      この場合, ここでリターン.
      ---------------------------------------- -}

	  if (_recursions > 0) {
	    --_recursions ;
	    return OM_OK ;
	  }
	
  {- -------------------------------------------
  (1) SimpleExit() を呼んでロックの開放処理を行う.
      ---------------------------------------- -}

	  void * List = _EntryList ;
	  SimpleExit (THREAD) ;
	
	  return OM_OK;
	}
	
```


