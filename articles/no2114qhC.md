---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/threadService.cpp

### 名前(function name)
```
StackFrameInfo::StackFrameInfo(javaVFrame* jvf, bool with_lock_info) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 各フィールドに値をセットする.
      ---------------------------------------- -}

	  _method = jvf->method();
	  _bci = jvf->bci();
	  _locked_monitors = NULL;

  {- -------------------------------------------
  (1) 「現在ロックされているオブジェクトモニターの一覧」も取得するようにと
      コンストラクタ引数で指定されていた場合 (= with_lock_info が true の場合), 
      javaVFrame::locked_monitors() でロックしているモニタの一覧を取得し, 
      #TODO
      ---------------------------------------- -}

	  if (with_lock_info) {
	    ResourceMark rm;
	    GrowableArray<MonitorInfo*>* list = jvf->locked_monitors();
	    int length = list->length();
	    if (length > 0) {
	      _locked_monitors = new (ResourceObj::C_HEAP) GrowableArray<oop>(length, true);
	      for (int i = 0; i < length; i++) {
	        MonitorInfo* monitor = list->at(i);
	        assert(monitor->owner(), "This monitor must have an owning object");
	        _locked_monitors->append(monitor->owner());
	      }
	    }
	  }
	}
	
```


