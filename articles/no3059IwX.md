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
     * If this thread was constructed using a separate
     * <code>Runnable</code> run object, then that
     * <code>Runnable</code> object's <code>run</code> method is called;
     * otherwise, this method does nothing and returns.
     * <p>
     * Subclasses of <code>Thread</code> should override this method.
     *
     * @see     #start()
     * @see     #stop()
     * @see     #Thread(ThreadGroup, Runnable, String)
     */
```

### 名前(function name)
```
    @Override
    public void run() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) target が設定されていれば, それを対象にして java.lang.Runnable.run() を呼び出す.
      target が設定されていなければ, 何もしない.
      ---------------------------------------- -}

	        if (target != null) {
	            target.run();
	        }
	    }
	
```


