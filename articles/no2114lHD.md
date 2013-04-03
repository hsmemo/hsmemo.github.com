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
     * Returns the total number of times that this sensor has been triggered.
     *
     * @return the total number of times that this sensor has been triggered.
     */
```

### 名前(function name)
```
    public long getCount() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) count フィールドの値をリターンするだけ.
      ---------------------------------------- -}

	        synchronized (lock) {
	            return count;
	        }
	    }
	
```


