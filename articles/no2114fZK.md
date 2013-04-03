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
    public long[] getThreadAllocatedBytes(long[] ids) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.ThreadImpl.getThreadAllocatedMemory1() を呼び出して, 
      引数で指定されたスレッドのメモリ確保量を取得する.
  
      ただし, スレッド毎のメモリ確保量計測機能がサポートされていない場合 
      (= verifyThreadAllocatedMemory() が false の場合)には, 
      単に -1 だけが詰まった long 配列を返すだけ.
      ---------------------------------------- -}

	        boolean verified = verifyThreadAllocatedMemory(ids);
	
	        long[] sizes = new long[ids.length];
	        java.util.Arrays.fill(sizes, -1);
	
	        if (verified) {
	            getThreadAllocatedMemory1(ids, sizes);
	        }
	        return sizes;
	    }
	
```


