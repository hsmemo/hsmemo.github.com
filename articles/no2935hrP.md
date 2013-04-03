---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
void JvmtiDeferredEventQueue::enqueue(const JvmtiDeferredEvent& event) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Service_lock->owned_by_self(), "Must own Service_lock");
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  process_pending_events();
	
  {- -------------------------------------------
  (1) 新しい QueueNode オブジェクトを作成し, 
      _queue_tail の先頭に追加する.
    
      (なお _queue_tail が空の場合は, _queue_tail だけでなく
       _queue_head も新しい QueueNode オブジェクトを指すようにする)
      ---------------------------------------- -}

	  // Events get added to the end of the queue (and are pulled off the front).
	  QueueNode* node = new QueueNode(event);
	  if (_queue_tail == NULL) {
	    _queue_tail = _queue_head = node;
	  } else {
	    assert(_queue_tail->next() == NULL, "Must be the last element in the list");
	    _queue_tail->set_next(node);
	    _queue_tail = node;
	  }
	
  {- -------------------------------------------
  (1) Service_lock に対して Monitor::notify_all() を呼び出して, ServiceThread を起床させる.
      ---------------------------------------- -}

	  Service_lock->notify_all();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert((_queue_head == NULL) == (_queue_tail == NULL),
	         "Inconsistent queue markers");
	}
	
```


