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
    void createGCNotification(long timestamp,
                              String gcName,
                              String gcAction,
                              String gcCause,
                              GcInfo gcInfo)  {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) もし通知を送信すべきリスナーが誰もいなければ, することはない. ここでリターン.
      ---------------------------------------- -}

	        if (!hasListeners()) {
	            return;
	        }
	
  {- -------------------------------------------
  (1) リスナーに送信する Notification オブジェクトを生成する.
      ---------------------------------------- -}

	        Notification notif = new Notification(GarbageCollectionNotificationInfo.GARBAGE_COLLECTION_NOTIFICATION,
	                                              getObjectName(),
	                                              getNextSeqNumber(),
	                                              timestamp,
	                                              gcName);

  {- -------------------------------------------
  (1) GC 情報を表す CompositeData を生成し, Notification オブジェクトに付加する.
      ---------------------------------------- -}

	        GarbageCollectionNotificationInfo info =
	            new GarbageCollectionNotificationInfo(gcName,
	                                                  gcAction,
	                                                  gcCause,
	                                                  gcInfo);
	
	        CompositeData cd =
	            GarbageCollectionNotifInfoCompositeData.toCompositeData(info);
	        notif.setUserData(cd);

  {- -------------------------------------------
  (1) sun.management.NotificationEmitterSupport.sendNotification() で
      リスナーに通知を送信する.
      ---------------------------------------- -}

	        sendNotification(notif);
	    }
	
```


