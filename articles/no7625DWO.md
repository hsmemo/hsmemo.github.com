---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/net/URLClassLoader.java
### 説明(description)

```
    /**
     * Finds and loads the class with the specified name from the URL search
     * path. Any URLs referring to JAR files are loaded and opened as needed
     * until the class is found.
     *
     * @param name the name of the class
     * @return the resulting class
     * @exception ClassNotFoundException if the class could not be found,
     *            or if the loader is closed.
     */
```

### 名前(function name)
```
    protected Class<?> findClass(final String name)
         throws ClassNotFoundException
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.net.URLClassLoader.defineClass() を呼んで, 
      name 引数で指定されたクラスをロードする.
      ---------------------------------------- -}

	        try {
	            return AccessController.doPrivileged(
	                new PrivilegedExceptionAction<Class>() {
	                    public Class run() throws ClassNotFoundException {
	                        String path = name.replace('.', '/').concat(".class");
	                        Resource res = ucp.getResource(path, false);
	                        if (res != null) {
	                            try {
	                                return defineClass(name, res);
	                            } catch (IOException e) {
	                                throw new ClassNotFoundException(name, e);
	                            }
	                        } else {
	                            throw new ClassNotFoundException(name);
	                        }
	                    }
	                }, acc);
	        } catch (java.security.PrivilegedActionException pae) {
	            throw (ClassNotFoundException) pae.getException();
	        }
	    }
	
```


