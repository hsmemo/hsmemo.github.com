---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.GarbageCollectorImpl の通知機能  
---
[Up](noCxwFq9be.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean クラスの処理 ： sun.management.GarbageCollectorImpl の通知機能  

--- 
## 概要(Summary)
GarbageCollectorImpl は NotificationEmitter 機能を有している.
これは GarbageCollectorImpl が MemoryManagerImpl のサブクラスであり, 
この MemoryManagerImpl が NotificationEmitterSupport のサブクラスであるため.

機能としては「GC 終了時に登録されているリスナーに通知を送る」というもの.
内部的には GCNotifier クラスと ServiceThread によって実現されている.

## 備考(Notes)
(なお, このメソッドは JSR-174 には存在しない Sun Microsystems の独自拡張機能)

## 処理の流れ (概要)(Execution Flows : Summary)
### リスナーの登録処理の流れ
```
sun.management.GarbageCollectorImpl.addNotificationListener()
-> sun.management.NotificationEmitterSupport.addNotificationListener()
-> sun.management.GarbageCollectorImpl.setNotificationEnabled()
   -> Java_sun_management_GarbageCollectorImpl_setNotificationEnabled()
      -> jmm_SetGCNotificationEnabled()
         -> GCMemoryManager::set_notification_enabled()
```

### リスナーへの通知送信処理の流れ
#### (1) 通知用のオブジェクトの作成と ServiceThread の起床処理
(GC 終了後に TraceMemoryManagerStats のデストラクタから呼び出される)

```
TraceMemoryManagerStats::~TraceMemoryManagerStats()
-> MemoryService::gc_end()
   -> GCMemoryManager::gc_end()
      (GCMemoryManager::is_notification_enabled() が true だと GCNotifier::pushNotification() が呼ばれる)
      -> GCNotifier::pushNotification()
         -> GCNotifier::addRequest()
            -> Monitor::notify_all()   (<= ServiceThread を起床させる)
```

#### (2) ServiceThread による GCNotifier の通知機能の呼び出し
```
ServiceThread::service_thread_entry()
-> GCNotifier::sendNotification()
   -> sun.management.GarbageCollectorImpl.createGCNotification()
      -> sun.management.NotificationEmitterSupport.sendNotification()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### sun.management.GarbageCollectorImpl.addNotificationListener()
See: [here](no2114JjA.html) for details
### sun.management.NotificationEmitterSupport.addNotificationListener()
(#Under Construction)

### Java_sun_management_GarbageCollectorImpl_setNotificationEnabled()
See: [here](no2114WtG.html) for details
### jmm_SetGCNotificationEnabled()
See: [here](no2114j3M.html) for details
### GCMemoryManager::set_notification_enabled()
See: [here](no2114wBT.html) for details

### GCNotifier::pushNotification()
See: [here](no21149LZ.html) for details
### GCNotifier::addRequest()
See: [here](no2114KWf.html) for details

### GCNotifier::sendNotification()
See: [here](no2114Xgl.html) for details
### sun.management.GarbageCollectorImpl.createGCNotification()
See: [here](no2114kqr.html) for details
### sun.management.NotificationEmitterSupport.sendNotification()
(#Under Construction)







