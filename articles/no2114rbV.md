---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/ThreadImpl.java

### 名前(function name)
```
    public long getCurrentThreadCpuTime() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.ThreadImpl.getThreadTotalCpuTime0() を呼び出し, 結果をリターンする.
  
      ただし, スレッドの CPU 時間計測機能がサポートされていない場合 
      (= verifyCurrentThreadCpuTime() が false の場合)には, 
      単に -1 を返すだけ.
      ---------------------------------------- -}

	        if (verifyCurrentThreadCpuTime()) {
	            return getThreadTotalCpuTime0(0);
	        }
	        return -1;
	    }
	
```


