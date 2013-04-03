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
     * Allocates a new {@code Thread} object. This constructor has the same
     * effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
     * {@code (group, null, name)}.
     *
     * @param  group
     *         the thread group. If {@code null} and there is a security
     *         manager, the group is determined by {@linkplain
     *         SecurityManager#getThreadGroup SecurityManager.getThreadGroup()}.
     *         If there is not a security manager or {@code
     *         SecurityManager.getThreadGroup()} returns {@code null}, the group
     *         is set to the current thread's thread group.
     *
     * @param  name
     *         the name of the new thread
     *
     * @throws  SecurityException
     *          if the current thread cannot create a thread in the specified
     *          thread group
     */
```

### 名前(function name)
```
    public Thread(ThreadGroup group, String name) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) java.lang.Thread.init() を呼び出すだけ.
      ---------------------------------------- -}

	        init(group, null, name, 0);
	    }
	
```


