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
    public long[] getThreadCpuTime(long[] ids) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下のどちらかの関数を呼び出して, 引数で指定されたスレッドの CPU 使用時間を取得する.
      * 指定されたスレッドが 1個しかない場合
        sun.management.ThreadImpl.getThreadTotalCpuTime0() を呼び出す.
      * 指定されたスレッドが 2個以上ある場合
        sun.management.ThreadImpl.getThreadTotalCpuTime1() を呼び出す.
  
      ただし, スレッドの CPU 時間計測機能がサポートされていない場合 
      (= verifyCurrentThreadCpuTime() が false の場合)には, 
      単に -1 だけが詰まった long 配列を返すだけ.
      ---------------------------------------- -}

	        boolean verified = verifyThreadCpuTime(ids);
	
	        int length = ids.length;
	        long[] times = new long[length];
	        java.util.Arrays.fill(times, -1);
	
	        if (verified) {
	            if (length == 1) {
	                long id = ids[0];
	                if (id == Thread.currentThread().getId()) {
	                    id = 0;
	                }
	                times[0] = getThreadTotalCpuTime0(id);
	            } else {
	                getThreadTotalCpuTime1(ids, times);
	            }
	        }
	        return times;
	    }
	
```


