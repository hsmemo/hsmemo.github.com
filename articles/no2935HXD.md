---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/serviceThread.cpp

### 名前(function name)
```
void ServiceThread::service_thread_entry(JavaThread* jt, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (以下の while ループが ServiceThread のメインループ)
      ---------------------------------------- -}

	  while (true) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    bool sensors_changed = false;
	    bool has_jvmti_events = false;
	    bool has_gc_notification_event = false;
	    JvmtiDeferredEvent jvmti_event;

    {- -------------------------------------------
  (1.1) 以下のどれかが true になるまで, 以下のブロック内で待機する
        (待機処理は, Service_lock に対して Monitor::wait() を呼ぶことで行う)
  
        * LowMemoryDetector::has_pending_requests()  (See: [here](no2114x0x.html) for details)
        * JvmtiDeferredEventQueue::has_events()      (See: [here](no3718UPQ.html) for details)
        * GCNotifier::has_event()                    (See: [here](no2114KPr.html) for details)
  
        (なお, JvmtiDeferredEventQueue::has_events() が true になることで待機が終了した場合, 
         ついでに JvmtiDeferredEventQueue::dequeue() を呼んで, 通知するイベントも取得している)
  
        (なお, Safepoint 処理用のコードがこのスレッドをちゃんと扱えるように, 
         この処理の間は ThreadBlockInVM で JavaThreadState を変更している)
        ---------------------------------------- -}

	    {
	      // Need state transition ThreadBlockInVM so that this thread
	      // will be handled by safepoint correctly when this thread is
	      // notified at a safepoint.
	
	      // This ThreadBlockInVM object is not also considered to be
	      // suspend-equivalent because ServiceThread is not visible to
	      // external suspension.
	
	      ThreadBlockInVM tbivm(jt);
	
	      MutexLockerEx ml(Service_lock, Mutex::_no_safepoint_check_flag);
	      while (!(sensors_changed = LowMemoryDetector::has_pending_requests()) &&
	             !(has_jvmti_events = JvmtiDeferredEventQueue::has_events()) &&
	              !(has_gc_notification_event = GCNotifier::has_event())) {
	        // wait until one of the sensors has pending requests, or there is a
	        // pending JVMTI event or JMX GC notification to post
	        Service_lock->wait(Mutex::_no_safepoint_check_flag);
	      }
	
	      if (has_jvmti_events) {
	        jvmti_event = JvmtiDeferredEventQueue::dequeue();
	      }
	    }
	
    {- -------------------------------------------
  (1.1) JvmtiDeferredEventQueue::has_events() が true になった場合は
        JvmtiDeferredEvent::post() を呼んでイベントの遅延通知を行う (See: [here](no3718UPQ.html) for details).
        ---------------------------------------- -}

	    if (has_jvmti_events) {
	      jvmti_event.post();
	    }
	
    {- -------------------------------------------
  (1.1) LowMemoryDetector::has_pending_requests() が true になった場合は
        LowMemoryDetector::process_sensor_changes() を呼んで閾値超過通知処理を行う (See: [here](no2114x0x.html) for details).
        ---------------------------------------- -}

	    if (sensors_changed) {
	      LowMemoryDetector::process_sensor_changes(jt);
	    }
	
    {- -------------------------------------------
  (1.1) GCNotifier::has_event() が true になった場合は
        GCNotifier::sendNotification() を呼んで GCNotifier の通知機能を呼び出す (See: [here](no2114KPr.html) for details).
        ---------------------------------------- -}

	    if(has_gc_notification_event) {
	        GCNotifier::sendNotification(CHECK);
	    }
	  }
	}
	
```


