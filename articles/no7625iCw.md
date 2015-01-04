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
     * Returns the system class loader for delegation.  This is the default
     * delegation parent for new <tt>ClassLoader</tt> instances, and is
     * typically the class loader used to start the application.
     *
     * <p> This method is first invoked early in the runtime's startup
     * sequence, at which point it creates the system class loader and sets it
     * as the context class loader of the invoking <tt>Thread</tt>.
     *
     * <p> The default system class loader is an implementation-dependent
     * instance of this class.
     *
     * <p> If the system property "<tt>java.system.class.loader</tt>" is defined
     * when this method is first invoked then the value of that property is
     * taken to be the name of a class that will be returned as the system
     * class loader.  The class is loaded using the default system class loader
     * and must define a public constructor that takes a single parameter of
     * type <tt>ClassLoader</tt> which is used as the delegation parent.  An
     * instance is then created using this constructor with the default system
     * class loader as the parameter.  The resulting class loader is defined
     * to be the system class loader.
     *
     * <p> If a security manager is present, and the invoker's class loader is
     * not <tt>null</tt> and the invoker's class loader is not the same as or
     * an ancestor of the system class loader, then this method invokes the
     * security manager's {@link
     * SecurityManager#checkPermission(java.security.Permission)
     * <tt>checkPermission</tt>} method with a {@link
     * RuntimePermission#RuntimePermission(String)
     * <tt>RuntimePermission("getClassLoader")</tt>} permission to verify
     * access to the system class loader.  If not, a
     * <tt>SecurityException</tt> will be thrown.  </p>
     *
     * @return  The system <tt>ClassLoader</tt> for delegation, or
     *          <tt>null</tt> if none
     *
     * @throws  SecurityException
     *          If a security manager exists and its <tt>checkPermission</tt>
     *          method doesn't allow access to the system class loader.
     *
     * @throws  IllegalStateException
     *          If invoked recursively during the construction of the class
     *          loader specified by the "<tt>java.system.class.loader</tt>"
     *          property.
     *
     * @throws  Error
     *          If the system property "<tt>java.system.class.loader</tt>"
     *          is defined but the named class could not be loaded, the
     *          provider class does not define the required constructor, or an
     *          exception is thrown by that constructor when it is invoked. The
     *          underlying cause of the error can be retrieved via the
     *          {@link Throwable#getCause()} method.
     *
     * @revised  1.4
     */
```

### 名前(function name)
```
    public static ClassLoader getSystemClassLoader() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.lang.ClassLoader.initSystemClassLoader() を呼んで, システムクラスローダを取得する
      (取得したシステムクラスローダは scl フィールドに格納される.
       取得に失敗したら, ここでリターン)
      ---------------------------------------- -}

	        initSystemClassLoader();
	        if (scl == null) {
	            return null;
	        }

  {- -------------------------------------------
  (1) もし SecurityManager が登録されていれば, チェックを行う.
      ---------------------------------------- -}

	        SecurityManager sm = System.getSecurityManager();
	        if (sm != null) {
	            ClassLoader ccl = getCallerClassLoader();
	            if (ccl != null && ccl != scl && !scl.isAncestor(ccl)) {
	                sm.checkPermission(SecurityConstants.GET_CLASSLOADER_PERMISSION);
	            }
	        }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	        return scl;
	    }
	
```


