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
     * Allocates a new {@code Thread} object. This constructor has the same
     * effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
     * {@code (null, null, name)}.
     *
     * @param   name
     *          the name of the new thread
     */
```

### 名前(function name)
```
    public Thread(String name) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.lang.Thread.init() を呼び出すだけ.
      ---------------------------------------- -}

	        init(null, null, name, 0);
	    }
	
```


