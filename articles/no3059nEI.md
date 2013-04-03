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
inline void ObjectMonitor::DequeueSpecificWaiter(ObjectWaiter* node) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(node != NULL, "should not dequeue NULL node");
	  assert(node->_prev != NULL, "node already removed from list");
	  assert(node->_next != NULL, "node already removed from list");

  {- -------------------------------------------
  (1) 引数で渡された ObjectWaiter (以下の node) を, 
      待ちキュー(_WaitSet)内から削除する.
      ---------------------------------------- -}

	  // when the waiter has woken up because of interrupt,
	  // timeout or other spurious wake-up, dequeue the
	  // waiter from waiting list
	  ObjectWaiter* next = node->_next;
	  if (next == node) {
	    assert(node->_prev == node, "invariant check");
	    _WaitSet = NULL;
	  } else {
	    ObjectWaiter* prev = node->_prev;
	    assert(prev->_next == node, "invariant check");
	    assert(next->_prev == node, "invariant check");
	    next->_prev = prev;
	    prev->_next = next;
	    if (_WaitSet == node) {
	      _WaitSet = next;
	    }
	  }
	  node->_next = NULL;
	  node->_prev = NULL;
	}
	
```


