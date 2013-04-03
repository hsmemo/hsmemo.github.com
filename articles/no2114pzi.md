---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp

### 名前(function name)
```
int Monitor::TryFast () {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まずは _LockWord フィールドの値が 0 だと想定し, 
      CASPTR で値を 0 から _LBIT に変更することを試みる.
  
      成功したら, ここでリターン (ロックが取れたので 1 をリターン)
      ---------------------------------------- -}

	  // Optimistic fast-path form ...
	  // Fast-path attempt for the common uncontended case.
	  // Avoid RTS->RTO $ coherence upgrade on typical SMP systems.
	  intptr_t v = CASPTR (&_LockWord, 0, _LBIT) ;  // agro ...
	  if (v == 0) return 1 ;
	
  {- -------------------------------------------
  (1) 以下, ロックを無事取得できるかあるいは(他人が取っているので)諦めるまで 
      次の for ループを繰り返す.
      
      ループ内での処理は以下の通り.
      (1) _LockWord の値を確認する. 
      (2) もし _LBIT ビットが既に立っていたら, (他のスレッドがロックを取っているので) 諦めてリターンする
          (ロックは取れなかったので 0 をリターン).
      (2) 逆に _LBIT がまだ立っていなければ, _LBIT ビットを立てた値を CASPTR で _LockWord に書き戻すことを試みる.
          成功したら, ここでリターン (ロックが取れたので 1 をリターン).
          失敗したら, _LockWord の値の確認に戻ってやり直し.
      ---------------------------------------- -}

	  for (;;) {
	    if ((v & _LBIT) != 0) return 0 ;
	    const intptr_t u = CASPTR (&_LockWord, v, v|_LBIT) ;
	    if (v == u) return 1 ;
	    v = u ;
	  }
	}
	
```


