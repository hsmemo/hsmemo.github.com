---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Thread.java
### 説明(description)

```
    /**
     * Initializes a Thread.
     *
     * @param g the Thread group
     * @param target the object whose run() method gets called
     * @param name the name of the new Thread
     * @param stackSize the desired stack size for the new thread, or
     *        zero to indicate that this parameter is to be ignored.
     */
```

### 名前(function name)
```
    private void init(ThreadGroup g, Runnable target, String name,
                      long stackSize) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数が null だったら, NullPointerException.
      ---------------------------------------- -}

	        if (name == null) {
	            throw new NullPointerException("name cannot be null");
	        }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	        Thread parent = currentThread();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	        SecurityManager security = System.getSecurityManager();
	        if (g == null) {
	            /* Determine if it's an applet or not */
	
	            /* If there is a security manager, ask the security manager
	               what to do. */
	            if (security != null) {
	                g = security.getThreadGroup();
	            }
	
	            /* If the security doesn't have a strong opinion of the matter
	               use the parent thread group. */
	            if (g == null) {
	                g = parent.getThreadGroup();
	            }
	        }
	
	        /* checkAccess regardless of whether or not threadgroup is
	           explicitly passed in. */
	        g.checkAccess();
	
	        /*
	         * Do we have the required permissions?
	         */
	        if (security != null) {
	            if (isCCLOverridden(getClass())) {
	                security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);
	            }
	        }
	
  {- -------------------------------------------
  (1) java.lang.ThreadGroup.addUnstarted() を呼び出し, 
      ---------------------------------------- -}

	        g.addUnstarted();
	
  {- -------------------------------------------
  (1) フィールドの初期化を行う
      ---------------------------------------- -}

	        this.group = g;
	        this.daemon = parent.isDaemon();
	        this.priority = parent.getPriority();
	        this.name = name.toCharArray();
	        if (security == null || isCCLOverridden(parent.getClass()))
	            this.contextClassLoader = parent.getContextClassLoader();
	        else
	            this.contextClassLoader = parent.contextClassLoader;
	        this.inheritedAccessControlContext = AccessController.getContext();
	        this.target = target;
	        setPriority(priority);
	        if (parent.inheritableThreadLocals != null)
	            this.inheritableThreadLocals =
	                ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);
	        /* Stash the specified stack size in case the VM cares */
	        this.stackSize = stackSize;
	
	        /* Set thread ID */
	        tid = nextThreadID();
	    }
	
```


