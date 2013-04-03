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
     * Clears this sensor.
     */
```

### 名前(function name)
```
    public void clear() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Sensor オブジェクトの状態を off にし,
      clearAction() メソッドを呼び出す.
      ---------------------------------------- -}

	        synchronized (lock) {
	            on = false;
	        }
	        clearAction();
	    }
	
```


