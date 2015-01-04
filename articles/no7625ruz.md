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
    /**
     * Returns a class loaded by the bootstrap class loader;
     * or return null if not found.
     */
```

### 名前(function name)
```
    private Class findBootstrapClassOrNull(String name)
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) name 引数がクラス名として不正な場合は, null をリターン.
      ---------------------------------------- -}

	        if (!checkName(name)) return null;
	
  {- -------------------------------------------
  (1) java.lang.ClassLoader.findBootstrapClass() を呼んで, 結果をリターン.
      ---------------------------------------- -}

	        return findBootstrapClass(name);
	    }
	
```


