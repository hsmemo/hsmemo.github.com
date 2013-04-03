---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/System.java

### 名前(function name)
```
    public static void exit(int status) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.lang.Runtime.exit() を呼ぶだけ.
      ---------------------------------------- -}

	        Runtime.getRuntime().exit(status);
	
```


