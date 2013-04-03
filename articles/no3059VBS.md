---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Thread.java
### 説明(description)

```
    /**
     * Returns the default handler invoked when a thread abruptly terminates
     * due to an uncaught exception. If the returned value is <tt>null</tt>,
     * there is no default.
     * @since 1.5
     * @see #setDefaultUncaughtExceptionHandler
     */
```

### 名前(function name)
```
    public static UncaughtExceptionHandler getDefaultUncaughtExceptionHandler(){
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) defaultUncaughtExceptionHandler フィールドの値をリターンするだけ.
      ---------------------------------------- -}

	        return defaultUncaughtExceptionHandler;
	    }
	
```


