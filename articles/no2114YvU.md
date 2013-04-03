---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/lowMemoryDetector.cpp

### 名前(function name)
```
void SensorInfo::process_pending_requests(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし未処理の通知がなければ, 何もすることはないので, ここでリターン.
      ---------------------------------------- -}

	  if (!has_pending_requests()) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) SensorInfo::trigger() または SensorInfo::clear() を呼び出す.
      (_pending_clear_count が 1以上なら SensorInfo::clear(), そうでなければ SensorInfo::trigger())
      ---------------------------------------- -}

	  int pending_count = pending_trigger_count();
	  if (pending_clear_count() > 0) {
	    clear(pending_count, CHECK);
	  } else {
	    trigger(pending_count, CHECK);
	  }
	
	}
	
```


