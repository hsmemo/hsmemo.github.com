---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
void Parker::unpark() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int s, status ;

  {- -------------------------------------------
  (1) _counter の値を 1 に変更する.
      (なお, この変更処理は os::Solaris::mutex_lock() で _mutex を取得して排他した状態で行う)
  
      もし, 変更前の _counter の値が 0 であれば, 
      (park() で寝ているスレッドがいるということなので)
      os::Solaris::cond_signal() でスレッドの起床処理を行う.
      ---------------------------------------- -}

	  status = os::Solaris::mutex_lock (_mutex) ;
	  assert (status == 0, "invariant") ;
	  s = _counter;
	  _counter = 1;
	  status = os::Solaris::mutex_unlock (_mutex) ;
	  assert (status == 0, "invariant") ;
	
	  if (s < 1) {
	    status = os::Solaris::cond_signal (_cond) ;
	    assert (status == 0, "invariant") ;
	  }
	}
	
```


