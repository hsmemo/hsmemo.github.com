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
     * Links the specified class.  This (misleadingly named) method may be
     * used by a class loader to link a class.  If the class <tt>c</tt> has
     * already been linked, then this method simply returns. Otherwise, the
     * class is linked as described in the "Execution" chapter of
     * <cite>The Java&trade; Language Specification</cite>.
     * </p>
     *
     * @param  c
     *         The class to link
     *
     * @throws  NullPointerException
     *          If <tt>c</tt> is <tt>null</tt>.
     *
     * @see  #defineClass(String, byte[], int, int)
     */
```

### 名前(function name)
```
    protected final void resolveClass(Class<?> c) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.lang.ClassLoader.resolveClass0() を呼び出すだけ.
      ---------------------------------------- -}

	        resolveClass0(c);
	    }
	
```


