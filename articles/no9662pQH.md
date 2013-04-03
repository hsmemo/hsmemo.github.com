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
// Caveat: TryLock() is not necessarily serializing if it returns failure.
// Callers must compensate as needed.

```

### 名前(function name)
```
int ObjectMonitor::TryLock (Thread * Self) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) _owner フィールドを確認した後, CAS によるロック確保を試みる (TATAS 方式).
      * 既に誰かにロックされていた場合 (_owner の値が NULL ではなかった場合)
        CAS するのは諦めて, 0 をリターンするだけ.
      * 誰にもロックされておらず, CAS も成功した場合.
        1 をリターン.
      * 確認時点ではロックされていなかったが, CAS が失敗した場合.
        0 をリターン.
        (コメントでは, 再挑戦してみてもいいが
         ちょうど今ロックが取られたばかりなので見込みは薄いだろう, 
         と書かれている.
         再挑戦用に, 一応コード自体は for ループで囲われているが...)
      ---------------------------------------- -}

	   for (;;) {
	      void * own = _owner ;
	      if (own != NULL) return 0 ;
	      if (Atomic::cmpxchg_ptr (Self, &_owner, NULL) == NULL) {
	         // Either guarantee _recursions == 0 or set _recursions = 0.
	         assert (_recursions == 0, "invariant") ;
	         assert (_owner == Self, "invariant") ;
	         // CONSIDER: set or assert that OwnerIsThread == 1
	         return 1 ;
	      }
	      // The lock had been free momentarily, but we lost the race to the lock.
	      // Interference -- the CAS failed.
	      // We can either return -1 or retry.
	      // Retry doesn't make as much sense because the lock was just acquired.
	      if (true) return -1 ;
	   }
	}
	
```


