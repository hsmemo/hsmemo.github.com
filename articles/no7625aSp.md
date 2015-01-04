---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/SecurityManager.java
### 説明(description)

```
    /**
     * Throws a <code>SecurityException</code> if the
     * calling thread is not allowed to access the package specified by
     * the argument.
     * <p>
     * This method is used by the <code>loadClass</code> method of class
     * loaders.
     * <p>
     * This method first gets a list of
     * restricted packages by obtaining a comma-separated list from
     * a call to
     * <code>java.security.Security.getProperty("package.access")</code>,
     * and checks to see if <code>pkg</code> starts with or equals
     * any of the restricted packages. If it does, then
     * <code>checkPermission</code> gets called with the
     * <code>RuntimePermission("accessClassInPackage."+pkg)</code>
     * permission.
     * <p>
     * If this method is overridden, then
     * <code>super.checkPackageAccess</code> should be called
     * as the first line in the overridden method.
     *
     * @param      pkg   the package name.
     * @exception  SecurityException  if the calling thread does not have
     *             permission to access the specified package.
     * @exception  NullPointerException if the package name argument is
     *             <code>null</code>.
     * @see        java.lang.ClassLoader#loadClass(java.lang.String, boolean)
     *  loadClass
     * @see        java.security.Security#getProperty getProperty
     * @see        #checkPermission(java.security.Permission) checkPermission
     */
```

### 名前(function name)
```
    public void checkPackageAccess(String pkg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        if (pkg == null) {
	            throw new NullPointerException("package name can't be null");
	        }
	
	        String[] pkgs;
	        synchronized (packageAccessLock) {
	            /*
	             * Do we need to update our property array?
	             */
	            if (!packageAccessValid) {
	                String tmpPropertyStr =
	                    AccessController.doPrivileged(
	                        new PrivilegedAction<String>() {
	                            public String run() {
	                                return java.security.Security.getProperty(
	                                    "package.access");
	                            }
	                        }
	                    );
	                packageAccess = getPackages(tmpPropertyStr);
	                packageAccessValid = true;
	            }
	
	            // Using a snapshot of packageAccess -- don't care if static field
	            // changes afterwards; array contents won't change.
	            pkgs = packageAccess;
	        }
	
	        /*
	         * Traverse the list of packages, check for any matches.
	         */
	        for (int i = 0; i < pkgs.length; i++) {
	            if (pkg.startsWith(pkgs[i]) || pkgs[i].equals(pkg + ".")) {
	                checkPermission(
	                    new RuntimePermission("accessClassInPackage."+pkg));
	                break;  // No need to continue; only need to check this once
	            }
	        }
	    }
	
```


