---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp
### 説明(description)

```
// after the thread acquires the lock in ::enter().  Equally, we could defer
// unlinking the thread until ::exit()-time.

```

### 名前(function name)
```
void ObjectMonitor::UnlinkAfterAcquire (Thread * Self, ObjectWaiter * SelfNode)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert (_owner == Self, "invariant") ;
	    assert (SelfNode->_thread == Self, "invariant") ;
	
  {- -------------------------------------------
  (1) カレントスレッドが EntryList 内にいる場合は (= TState が ObjectWaiter::TS_ENTER の場合は), 
      EntryList 内から対応する ObjectWaiter を除去する.
      ---------------------------------------- -}

	    if (SelfNode->TState == ObjectWaiter::TS_ENTER) {
	        // Normal case: remove Self from the DLL EntryList .
	        // This is a constant-time operation.
	        ObjectWaiter * nxt = SelfNode->_next ;
	        ObjectWaiter * prv = SelfNode->_prev ;
	        if (nxt != NULL) nxt->_prev = prv ;
	        if (prv != NULL) prv->_next = nxt ;
	        if (SelfNode == _EntryList ) _EntryList = nxt ;
	        assert (nxt == NULL || nxt->TState == ObjectWaiter::TS_ENTER, "invariant") ;
	        assert (prv == NULL || prv->TState == ObjectWaiter::TS_ENTER, "invariant") ;
	        TEVENT (Unlink from EntryList) ;

  {- -------------------------------------------
  (1) カレントスレッドが cxq 内にいる場合は, 
      cxq 内から対応する ObjectWaiter を除去する.
      ---------------------------------------- -}

	    } else {
	        guarantee (SelfNode->TState == ObjectWaiter::TS_CXQ, "invariant") ;
	        // Inopportune interleaving -- Self is still on the cxq.
	        // This usually means the enqueue of self raced an exiting thread.
	        // Normally we'll find Self near the front of the cxq, so
	        // dequeueing is typically fast.  If needbe we can accelerate
	        // this with some MCS/CHL-like bidirectional list hints and advisory
	        // back-links so dequeueing from the interior will normally operate
	        // in constant-time.
	        // Dequeue Self from either the head (with CAS) or from the interior
	        // with a linear-time scan and normal non-atomic memory operations.
	        // CONSIDER: if Self is on the cxq then simply drain cxq into EntryList
	        // and then unlink Self from EntryList.  We have to drain eventually,
	        // so it might as well be now.
	
	        ObjectWaiter * v = _cxq ;
	        assert (v != NULL, "invariant") ;
	        if (v != SelfNode || Atomic::cmpxchg_ptr (SelfNode->_next, &_cxq, v) != v) {
	            // The CAS above can fail from interference IFF a "RAT" arrived.
	            // In that case Self must be in the interior and can no longer be
	            // at the head of cxq.
	            if (v == SelfNode) {
	                assert (_cxq != v, "invariant") ;
	                v = _cxq ;          // CAS above failed - start scan at head of list
	            }
	            ObjectWaiter * p ;
	            ObjectWaiter * q = NULL ;
	            for (p = v ; p != NULL && p != SelfNode; p = p->_next) {
	                q = p ;
	                assert (p->TState == ObjectWaiter::TS_CXQ, "invariant") ;
	            }
	            assert (v != SelfNode,  "invariant") ;
	            assert (p == SelfNode,  "Node not found on cxq") ;
	            assert (p != _cxq,      "invariant") ;
	            assert (q != NULL,      "invariant") ;
	            assert (q->_next == p,  "invariant") ;
	            q->_next = p->_next ;
	        }
	        TEVENT (Unlink from cxq) ;
	    }
	
  {- -------------------------------------------
  (1) カレントスレッドに対応する ObjectWaiter のフィールドをリセットしておく.
      (コメントを読む限り, リセットしなくても統計情報以外に影響はなさそうだが...)
      ---------------------------------------- -}

	    // Diagnostic hygiene ...
	    SelfNode->_prev  = (ObjectWaiter *) 0xBAD ;
	    SelfNode->_next  = (ObjectWaiter *) 0xBAD ;
	    SelfNode->TState = ObjectWaiter::TS_RUN ;
	}
	
```


