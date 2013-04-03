---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/Sensor.java
### 説明(description)

```
    /**
     * Triggers this sensor piggybacking a memory usage object.
     * This method sets this sensor on
     * and increments the count with the input <tt>increment</tt>.
     */
```

### 名前(function name)
```
    public void trigger(int increment, MemoryUsage usage) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Sensor オブジェクトの状態を on にし, 引数での指定分だけカウンタをインクリメントした後.
      triggerAction() メソッドを呼び出してリスナーへの通知処理を行う.
      ---------------------------------------- -}

	        synchronized (lock) {
	            on = true;
	            count += increment;
	            // Do something here...
	        }
	        triggerAction(usage);
	    }
	
```


