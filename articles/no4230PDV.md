---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/synchronizer.cpp

### 名前(function name)
```
void ObjectSynchronizer::fast_exit(oop object, BasicLock* lock, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      (ここに biased の状態でくることはないはず)
      ---------------------------------------- -}

	  assert(!object->mark()->has_bias_pattern(), "should not see bias pattern here");

  {- -------------------------------------------
  (1) もし BasicObjectLock の displaced_header が NULL なら (= 自分が複数回ロックを取得していたケースでは), 
      ここでリターン.
  
      (なお, if の中で色々やっているようにも見えるが, 全部 assert チェック)
      ---------------------------------------- -}

	  // if displaced header is null, the previous enter is recursive enter, no-op
	  markOop dhw = lock->displaced_header();
	  markOop mark ;
	  if (dhw == NULL) {
	     // Recursive stack-lock.
	     // Diagnostics -- Could be: stack-locked, inflating, inflated.
	     mark = object->mark() ;
	     assert (!mark->is_neutral(), "invariant") ;
	     if (mark->has_locker() && mark != markOopDesc::INFLATING()) {
	        assert(THREAD->is_lock_owned((address)mark->locker()), "invariant") ;
	     }
	     if (mark->has_monitor()) {
	        ObjectMonitor * m = mark->monitor() ;
	        assert(((oop)(m->object()))->mark() == mark, "invariant") ;
	        assert(m->is_entered(THREAD), "invariant") ;
	     }
	     return ;
	  }
	
  {- -------------------------------------------
  (1) もし stack-locked であれば CAS で mark を書き戻す. 
      成功したらここでリターン.
      ---------------------------------------- -}

	  mark = object->mark() ;
	
	  // If the object is stack-locked by the current thread, try to
	  // swing the displaced header from the box back to the mark.
	  if (mark == (markOop) lock) {
	     assert (dhw->is_neutral(), "invariant") ;
	     if ((markOop) Atomic::cmpxchg_ptr (dhw, object->mark_addr(), mark) == mark) {
	        TEVENT (fast_exit: release stacklock) ;
	        return;
	     }
	  }
	
  {- -------------------------------------------
  (1) 上記のどれでもなければ (あるいは CAS が失敗した場合は), 
      ObjectSynchronizer::inflate() で ObjectMonitor オブジェクトを取得した後, 
      ObjectMonitor::exit() でアンロック処理を行う.
      ---------------------------------------- -}

	  ObjectSynchronizer::inflate(THREAD, object)->exit (THREAD) ;
	}
	
```


