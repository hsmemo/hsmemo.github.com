---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Shutdown.java
### 説明(description)

```
    /* The halt method is synchronized on the halt lock
     * to avoid corruption of the delete-on-shutdown file list.
     * It invokes the true native halt method.
     */
```

### 名前(function name)
```
    static void halt(int status) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 排他を取った状態で Java_java_lang_Shutdown_halt0() を呼び出すだけ.
      ---------------------------------------- -}

	        synchronized (haltLock) {
	            halt0(status);
	        }
	
```


