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
     * Converts an array of bytes into an instance of class <tt>Class</tt>,
     * with an optional <tt>ProtectionDomain</tt>.  If the domain is
     * <tt>null</tt>, then a default domain will be assigned to the class as
     * specified in the documentation for {@link #defineClass(String, byte[],
     * int, int)}.  Before the class can be used it must be resolved.
     *
     * <p> The first class defined in a package determines the exact set of
     * certificates that all subsequent classes defined in that package must
     * contain.  The set of certificates for a class is obtained from the
     * {@link java.security.CodeSource <tt>CodeSource</tt>} within the
     * <tt>ProtectionDomain</tt> of the class.  Any classes added to that
     * package must contain the same set of certificates or a
     * <tt>SecurityException</tt> will be thrown.  Note that if
     * <tt>name</tt> is <tt>null</tt>, this check is not performed.
     * You should always pass in the <a href="#name">binary name</a> of the
     * class you are defining as well as the bytes.  This ensures that the
     * class you are defining is indeed the class you think it is.
     *
     * <p> The specified <tt>name</tt> cannot begin with "<tt>java.</tt>", since
     * all classes in the "<tt>java.*</tt> packages can only be defined by the
     * bootstrap class loader.  If <tt>name</tt> is not <tt>null</tt>, it
     * must be equal to the <a href="#name">binary name</a> of the class
     * specified by the byte array "<tt>b</tt>", otherwise a {@link
     * <tt>NoClassDefFoundError</tt>} will be thrown.  </p>
     *
     * @param  name
     *         The expected <a href="#name">binary name</a> of the class, or
     *         <tt>null</tt> if not known
     *
     * @param  b
     *         The bytes that make up the class data. The bytes in positions
     *         <tt>off</tt> through <tt>off+len-1</tt> should have the format
     *         of a valid class file as defined by
     *         <cite>The Java&trade; Virtual Machine Specification</cite>.
     *
     * @param  off
     *         The start offset in <tt>b</tt> of the class data
     *
     * @param  len
     *         The length of the class data
     *
     * @param  protectionDomain
     *         The ProtectionDomain of the class
     *
     * @return  The <tt>Class</tt> object created from the data,
     *          and optional <tt>ProtectionDomain</tt>.
     *
     * @throws  ClassFormatError
     *          If the data did not contain a valid class
     *
     * @throws  NoClassDefFoundError
     *          If <tt>name</tt> is not equal to the <a href="#name">binary
     *          name</a> of the class specified by <tt>b</tt>
     *
     * @throws  IndexOutOfBoundsException
     *          If either <tt>off</tt> or <tt>len</tt> is negative, or if
     *          <tt>off+len</tt> is greater than <tt>b.length</tt>.
     *
     * @throws  SecurityException
     *          If an attempt is made to add this class to a package that
     *          contains classes that were signed by a different set of
     *          certificates than this class, or if <tt>name</tt> begins with
     *          "<tt>java.</tt>".
     */
```

### 名前(function name)
```
    protected final Class<?> defineClass(String name, byte[] b, int off, int len,
                                         ProtectionDomain protectionDomain)
        throws ClassFormatError
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) preDefineClass() を呼んで, 使用する protection domain を調整する.
      また, name 引数で指定されたクラスが, ロード対象として不正ではないかどうかのチェックも行う.
      (不正な場合は, 例外が送出される)
      ---------------------------------------- -}

	        protectionDomain = preDefineClass(name, protectionDomain);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	        Class c = null;
	        String source = defineClassSourceLocation(protectionDomain);
	
  {- -------------------------------------------
  (1) defineClass1() を呼んで, ロード処理を行う.
  
      なお, もし ClassFormatError になった場合は, 
      defineTransformedClass() でのロード処理を試みる.
      (このメソッドでは, 登録されている ClassFileTransformer でバイトコードを変更してからロードを行う)
      ---------------------------------------- -}

	        try {
	            c = defineClass1(name, b, off, len, protectionDomain, source);
	        } catch (ClassFormatError cfe) {
	            c = defineTransformedClass(name, b, off, len, protectionDomain, cfe,
	                                       source);
	        }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        postDefineClass(c, protectionDomain);

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	        return c;
	    }
	
```


