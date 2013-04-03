---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.inline.hpp

### 名前(function name)
```
inline void JvmtiEnvThreadEventEnable::set_user_enabled(jvmtiEvent event_type, bool enabled) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _event_user_enabled フィールドの JvmtiEventEnabled オブジェクトに対して, 
      event_type 引数で指定されたイベント種別の有効／無効を変更する.
      ---------------------------------------- -}

	  _event_user_enabled.set_enabled(event_type, enabled);
	}
	
```


