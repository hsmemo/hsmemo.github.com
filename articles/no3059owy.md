---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp

### 名前(function name)
```
inline void ObjectMonitor::AddWaiter(ObjectWaiter* node) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(node != NULL, "should not dequeue NULL node");
	  assert(node->_prev == NULL, "node already in list");
	  assert(node->_next == NULL, "node already in list");

  {- -------------------------------------------
  (1) 引数で渡された ObjectWaiter (以下の node) を, 
      待ちキュー(_WaitSet) の末尾に追加する.
      ---------------------------------------- -}

	  // put node at end of queue (circular doubly linked list)
	  if (_WaitSet == NULL) {
	    _WaitSet = node;
	    node->_prev = node;
	    node->_next = node;
	  } else {
	    ObjectWaiter* head = _WaitSet ;
	    ObjectWaiter* tail = head->_prev;
	    assert(tail->_next == head, "invariant check");
	    tail->_next = node;
	    head->_prev = node;
	    node->_next = head;
	    node->_prev = tail;
	  }
	}
	
```


