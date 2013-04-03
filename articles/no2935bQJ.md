---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp

### 名前(function name)
```
void
JvmtiEventControllerPrivate::change_field_watch(jvmtiEvent event_type, bool added) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int *count_addr;
	
  {- -------------------------------------------
  (1) 指定された event_type に対応する JvmtiExport 内のフィールドを取得する.
      ---------------------------------------- -}

	  switch (event_type) {
	  case JVMTI_EVENT_FIELD_MODIFICATION:
	    count_addr = (int *)JvmtiExport::get_field_modification_count_addr();
	    break;
	  case JVMTI_EVENT_FIELD_ACCESS:
	    count_addr = (int *)JvmtiExport::get_field_access_count_addr();
	    break;
	  default:
	    assert(false, "incorrect event");
	    return;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  EC_TRACE(("JVMTI [-] # change field watch - %s %s count=%d",
	            event_type==JVMTI_EVENT_FIELD_MODIFICATION? "modification" : "access",
	            added? "add" : "remove",
	            *count_addr));
	
  {- -------------------------------------------
  (1) 取得したフィールドに対して, 
      added 引数(追加かクリアか)に応じてインクリメント/デクリメントを行う.
  
      (必要があれば, JvmtiEventControllerPrivate::recompute_enabled() を呼んで, 
      "truly enabled event" 情報を更新する)
      ---------------------------------------- -}

	  if (added) {
	    (*count_addr)++;
	    if (*count_addr == 1) {
	      recompute_enabled();
	    }
	  } else {
	    if (*count_addr > 0) {
	      (*count_addr)--;
	      if (*count_addr == 0) {
	        recompute_enabled();
	      }
	    } else {
	      assert(false, "field watch out of phase");
	    }
	  }
	}
	
```


