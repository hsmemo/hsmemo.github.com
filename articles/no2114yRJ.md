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
     * Tests if this sensor is currently on.
     *
     * @return <tt>true</tt> if the sensor is currently on;
     *         <tt>false</tt> otherwise.
     *
     */
```

### 名前(function name)
```
    public boolean isOn() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) on フィールドの値をリターンするだけ.
      ---------------------------------------- -}

	        synchronized (lock) {
	            return on;
	        }
	    }
	
```


