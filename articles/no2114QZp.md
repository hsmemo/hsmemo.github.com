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
static int ParkCommon (ParkEvent * ev, jlong timo) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で指定されたタイムアウト時間(以下の timo)を微修正する.
      (NativeMonitorTimeout オプションに 0 より大きい値がセットされており, かつ
       引数の timo が NativeMonitorTimeout を超えているかもしくは 0 以下の場合には, 
       引数の timo の代わりに NativeMonitorTimeout の値を timo として用いる.)
      ---------------------------------------- -}

	  // Diagnostic support - periodically unwedge blocked threads
	  intx nmt = NativeMonitorTimeout ;
	  if (nmt > 0 && (nmt < timo || timo <= 0)) {
	     timo = nmt ;
	  }

  {- -------------------------------------------
  (1) os::PlatformEvent::park() を呼び出して眠りにつく.
      眠りから覚めたら結果をリターン.
  
      なお, タイムアウト時間が指定されているかどうかに応じて, 呼び出し先や結果が少し変わる.
      * タイムアウト時間が指定されていない場合 (= timo が 0 の場合): 
        os::PlatformEvent::park() を呼び出す.
        この場合の結果は, 常に OS_OK.
      * タイムアウト時間が指定されている場合 (= timo が 0 ではない場合): 
        os::PlatformEvent::park(jlong millis) を呼び出す
        この場合の結果は, os::PlatformEvent::park(jlong millis) の返値.
      ---------------------------------------- -}

	  int err = OS_OK ;
	  if (0 == timo) {
	    ev->park() ;
	  } else {
	    err = ev->park(timo) ;
	  }
	  return err ;
	}
	
```


