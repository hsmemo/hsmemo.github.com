---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/gcNotifier.cpp

### 名前(function name)
```
void GCNotifier::addRequest(GCNotificationRequest *request) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (Service_lock に対して notify_all() を呼び出すので, Service_lock のロックを取っておく)
      ---------------------------------------- -}

	  MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);

  {- -------------------------------------------
  (1) 新しい GCNotificationRequest オブジェクトをリストにつないでおく.
      ---------------------------------------- -}

	  if(first_request == NULL) {
	    first_request = request;
	  } else {
	    last_request->next = request;
	  }
	  last_request = request;

  {- -------------------------------------------
  (1) Service_lock に対して Monitor::notify_all() を呼び出して, ServiceThread を起床させる.
      ---------------------------------------- -}

	  Service_lock->notify_all();
	}
	
```


