---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/management/GarbageCollectorImpl.java

### 名前(function name)
```
    public synchronized void addNotificationListener(NotificationListener listener,
                                                     NotificationFilter filter,
                                                     Object handback)
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) sun.management.NotificationEmitterSupport.addNotificationListener() で, 
      リスナーの登録処理を行う.
  
      (ついでに, 登録処理の前後でリスナーの有無を調べておく)
      ---------------------------------------- -}

	        boolean before = hasListeners();
	        super.addNotificationListener(listener, filter, handback);
	        boolean after = hasListeners();

  {- -------------------------------------------
  (1) もし, これまではリスナーがいなかったが, 今回の登録処理でリスナーが存在するようになった場合, 
      sun.management.GarbageCollectorImpl.setNotificationEnabled() を呼んで
      リスナーへの通知処理を行うようにしておく.
      ---------------------------------------- -}

	        if (!before && after) {
	            setNotificationEnabled(this, true);
	        }
	    }
	
```


