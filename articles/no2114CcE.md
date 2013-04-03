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
inline int Monitor::AcquireOrPush (ParkEvent * ESelf) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  intptr_t v = _LockWord.FullWord ;

  {- -------------------------------------------
  (1) 以下, ロックを無事取得できるかあるいは待ち行列に自分を追加するまで 
      次の for ループを繰り返す.
      
      ループ内での処理は以下の通り.
      (1) _LockWord の値を確認する. 
      (2) もし _LBIT がまだ立っていなければ, _LBIT ビットを立てた値を CASPTR で _LockWord に書き戻すことを試みる.
          成功したら, ここでリターン (ロックが取れたので 1 をリターン).
          失敗したら _LockWord の値の確認に戻ってやり直し.
      (2) 逆に _LBIT ビットが既に立っていたら, (他のスレッドがロックを取っているので) 
          引数の ParkEvent オブジェクト(ESelf) を _LockWord に追加することを試みる.
          (より具体的に言うと, 既存の _LockWord の値(から _LBIT ビットを消したもの)を ESelf->ListNext につないだ後, 
           CASPTR で _LockWord を ESelf(に _LBIT ビットを立てたもの) に書き換える.)
          成功したら, ここでリターン (ロックは取れなかったので 0 をリターン).
          失敗したら _LockWord の値の確認に戻ってやり直し.
      ---------------------------------------- -}

	  for (;;) {
	    if ((v & _LBIT) == 0) {
	      const intptr_t u = CASPTR (&_LockWord, v, v|_LBIT) ;
	      if (u == v) return 1 ;        // indicate acquired
	      v = u ;
	    } else {
	      // Anticipate success ...
	      ESelf->ListNext = (ParkEvent *) (v & ~_LBIT) ;
	      const intptr_t u = CASPTR (&_LockWord, v, intptr_t(ESelf)|_LBIT) ;
	      if (u == v) return 0 ;        // indicate pushed onto cxq
	      v = u ;
	    }
	    // Interference - LockWord change - just retry
	  }
	}
	
```


