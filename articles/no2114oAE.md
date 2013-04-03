---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp
### 説明(description)


```
// Currently notifyAll() transfers the waiters one-at-a-time from the waitset
// to the cxq.  This could be done more efficiently with a single bulk en-mass transfer,
// but in practice notifyAll() for large #s of threads is rare and not time-critical.
// Beware too, that we invert the order of the waiters.  Lets say that the
// waitset is "ABCD" and the cxq is "XYZ".  After a notifyAll() the waitset
// will be empty and the cxq will be "DCBAXYZ".  This is benign, of course.
```

### 名前(function name)
```
bool Monitor::notify_all() {
```

### 本体部(body)
```
	(コメントによると, 
	 現状の Monitor::notify_all() ではスレッドを一つずつ notify() している.
	 まとめて処理するように変えればさらに効率よくできるけど, 
	 そもそもたくさんのスレッドが wait しているケースが稀だし, あんまり性能上重要でもない.
	
	 ついでに, waitset に並んでいた順番と cxq に並んだときの順番が逆になるけど, 
	 これも当然ながらたいした問題じゃない, 
	 とのこと.)
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert (_owner == Thread::current(), "invariant") ;
	  assert (ILocked(), "invariant") ;

  {- -------------------------------------------
  (1) _WaitSet が空になるまで Monitor::notify() を呼び続けるだけ.
      ---------------------------------------- -}

	  while (_WaitSet != NULL) notify() ;
	  return true ;
	}
	
```


