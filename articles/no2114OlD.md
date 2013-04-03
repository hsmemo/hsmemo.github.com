---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp

### 名前(function name)
```
void os::PlatformEvent::unpark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee (_ParkHandle != NULL, "Invariant") ;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int v ;

  {- -------------------------------------------
  (1) _Event の値が 0 より大きければ (= つまり 1 であれば), ここでリターンする.
      (なお, このプロセッサの store buffer 内の値を見てしまう恐れがあるので, 
       OrderAccess::fence() を張って確認している)
    
      そうでなければ Atomic::cmpxchg() で _Event の値を 1 増加させておく.
      (Atomic::cmpxchg() が失敗した場合は, 成功するまで以上の処理を繰り返す)
      ---------------------------------------- -}

	  for (;;) {
	      v = _Event ;      // Increment _Event if it's < 1.
	      if (v > 0) {
	         // If it's already signaled just return.
	         // The LD of _Event could have reordered or be satisfied
	         // by a read-aside from this processor's write buffer.
	         // To avoid problems execute a barrier and then
	         // ratify the value.  A degenerate CAS() would also work.
	         // Viz., CAS (v+0, &_Event, v) == v).
	         OrderAccess::fence() ;
	         if (_Event == v) return ;
	         continue ;
	      }
	      if (Atomic::cmpxchg (v+1, &_Event, v) == v) break ;
	  }

  {- -------------------------------------------
  (1) 変更前の _Event の値が負値だった場合は
      park() で待機しているスレッドがいる(かもしれない)ので, 
      SetEvent() で起こしてやる.
      ---------------------------------------- -}

	  if (v < 0) {
	     ::SetEvent (_ParkHandle) ;
	  }
	}
	
```


