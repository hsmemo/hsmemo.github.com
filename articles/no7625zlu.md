---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/misc/Launcher.java
### 説明(description)
package のチェックを行う処理を追加しただけの URLClassLoader.loadClass() のラッパー.

```
        /**
         * Override loadClass so we can checkPackageAccess.
         */
```

### 名前(function name)
```
        public Class loadClass(String name, boolean resolve)
            throws ClassNotFoundException
        {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) checkPackageAccess() で package のチェックを行っておく.
      ---------------------------------------- -}

	            int i = name.lastIndexOf('.');
	            if (i != -1) {
	                SecurityManager sm = System.getSecurityManager();
	                if (sm != null) {
	                    sm.checkPackageAccess(name.substring(0, i));
	                }
	            }

  {- -------------------------------------------
  (1) スーパークラスの loadClass() に処理を委譲.
      ---------------------------------------- -}

	            return (super.loadClass(name, resolve));
	        }
	
```


