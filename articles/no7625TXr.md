---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/ClassLoader.java
### 説明(description)

```
    // Invoked by the VM to record every loaded class with this loader.
```

### 名前(function name)
```
    void addClass(Class c) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) c 引数で指定された Class オブジェクトを
      classes フィールドの Vector に追加する.
      ---------------------------------------- -}

	        classes.addElement(c);
	    }
	
```


