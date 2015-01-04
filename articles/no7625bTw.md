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
     * Loads the class with the specified <a href="#name">binary name</a>.  The
     * default implementation of this method searches for classes in the
     * following order:
     *
     * <p><ol>
     *
     *   <li><p> Invoke {@link #findLoadedClass(String)} to check if the class
     *   has already been loaded.  </p></li>
     *
     *   <li><p> Invoke the {@link #loadClass(String) <tt>loadClass</tt>} method
     *   on the parent class loader.  If the parent is <tt>null</tt> the class
     *   loader built-in to the virtual machine is used, instead.  </p></li>
     *
     *   <li><p> Invoke the {@link #findClass(String)} method to find the
     *   class.  </p></li>
     *
     * </ol>
     *
     * <p> If the class was found using the above steps, and the
     * <tt>resolve</tt> flag is true, this method will then invoke the {@link
     * #resolveClass(Class)} method on the resulting <tt>Class</tt> object.
     *
     * <p> Subclasses of <tt>ClassLoader</tt> are encouraged to override {@link
     * #findClass(String)}, rather than this method.  </p>
     *
     * <p> Unless overridden, this method synchronizes on the result of
     * {@link #getClassLoadingLock <tt>getClassLoadingLock</tt>} method
     * during the entire class loading process.
     *
     * @param  name
     *         The <a href="#name">binary name</a> of the class
     *
     * @param  resolve
     *         If <tt>true</tt> then resolve the class
     *
     * @return  The resulting <tt>Class</tt> object
     *
     * @throws  ClassNotFoundException
     *          If the class could not be found
     */
```

### 名前(function name)
```
    protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        synchronized (getClassLoadingLock(name)) {

  {- -------------------------------------------
  (1) findLoadedClass() を呼んで, 既にロード済みかどうかをチェックする (ロード済みならそれを使う)
      ---------------------------------------- -}

	            // First, check if the class has already been loaded
	            Class c = findLoadedClass(name);

  {- -------------------------------------------
  (1) まだロードしていなければ, 以下の if ブロック内でロードを行う.
      ---------------------------------------- -}

	            if (c == null) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	                long t0 = System.nanoTime();

    {- -------------------------------------------
  (1.1) 親のクラスローダーがいれば, 親の loadClass() でロードを試みる.
        親がいなければ, findBootstrapClassOrNull() でロードを試みる.
        
        (なお, システムクラスローダーの親は ExtClassLoader である模様)
        ---------------------------------------- -}

	                try {
	                    if (parent != null) {
	                        c = parent.loadClass(name, false);
	                    } else {
	                        c = findBootstrapClassOrNull(name);
	                    }
	                } catch (ClassNotFoundException e) {
	                    // ClassNotFoundException thrown if class not found
	                    // from the non-null parent class loader
	                }
	
    {- -------------------------------------------
  (1.1) 以上の処理でロードできなければ, findClass() で探す.
        ---------------------------------------- -}

	                if (c == null) {
	                    // If still not found, then invoke findClass in order
	                    // to find the class.
	                    long t1 = System.nanoTime();
	                    c = findClass(name);
	
	                    // this is the defining class loader; record the stats
	                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
	                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
	                    sun.misc.PerfCounter.getFindClasses().increment();
	                }
	            }

  {- -------------------------------------------
  (1) もし引数で resolve まで行うように指定されていれば, 
      java.lang.ClassLoader.resolveClass() を呼んで, この場で resolve する.
      ---------------------------------------- -}

	            if (resolve) {
	                resolveClass(c);
	            }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	            return c;
	        }
	    }
	
```


