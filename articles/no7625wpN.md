---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/misc/Launcher.java
### 説明(description)

```
    /*
     * Returns the class loader used to launch the main application.
     */
```

### 名前(function name)
```
    public ClassLoader getClassLoader() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (単なる getter method)
      (このフィールドには sun.misc.Launcher$AppClassLoader.getAppClassLoader() の返値が入っている. 値はコンストラクタで設定される)
      ---------------------------------------- -}

	        return loader;
	    }
	
```


