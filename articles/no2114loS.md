---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)

```
// Used to convert frequent JVM_Yield() to nops
```

### 名前(function name)
```
bool os::dont_yield() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) DontYieldALot オプションが指定されていなければ false をリターンするだけ.
      指定されている場合には, 最後に yield したときからの経過時間に応じて決める.
      (DontYieldALotInterval(ミリ秒) 以上経過していれば false, そうでなければ true)
      ---------------------------------------- -}

	  if (DontYieldALot) {
	    static hrtime_t last_time = 0;
	    hrtime_t diff = getTimeNanos() - last_time;
	
	    if (diff < DontYieldALotInterval * 1000000)
	      return true;
	
	    last_time += diff;
	
	    return false;
	  }
	  else {
	    return false;
	  }
	}
	
```


