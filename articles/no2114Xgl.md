---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/gcNotifier.cpp

### 名前(function name)
```
void GCNotifier::sendNotification(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);

  {- -------------------------------------------
  (1) 通知オブジェクト(GCNotificationRequest)がないのであれば, 何もしない
      ---------------------------------------- -}

	  GCNotificationRequest *request = getRequest();
	  if(request != NULL) {

  {- -------------------------------------------
  (1) GCNotifier::createGcInfo() で
      GCStatInfo オブジェクトから com.sun.management.GcInfo オブジェクトを生成.
      ---------------------------------------- -}

	    Handle objGcInfo = createGcInfo(request->gcManager,request->gcStatInfo,THREAD);
	
  {- -------------------------------------------
  (1) sun.management.GarbageCollectorImpl.createGCNotification() を呼び出す.
      ---------------------------------------- -}

	    Handle objName = java_lang_String::create_from_platform_dependent_str(request->gcManager->name(), CHECK);
	    Handle objAction = java_lang_String::create_from_platform_dependent_str(request->gcAction, CHECK);
	    Handle objCause = java_lang_String::create_from_platform_dependent_str(request->gcCause, CHECK);
	
	    klassOop k = Management::sun_management_GarbageCollectorImpl_klass(CHECK);
	    instanceKlassHandle gc_mbean_klass (THREAD, k);
	
	    instanceOop gc_mbean = request->gcManager->get_memory_manager_instance(THREAD);
	    instanceHandle gc_mbean_h(THREAD, gc_mbean);
	    if (!gc_mbean_h->is_a(k)) {
	      THROW_MSG(vmSymbols::java_lang_IllegalArgumentException(),
	                "This GCMemoryManager doesn't have a GarbageCollectorMXBean");
	    }
	
	    JavaValue result(T_VOID);
	    JavaCallArguments args(gc_mbean_h);
	    args.push_long(request->timestamp);
	    args.push_oop(objName);
	    args.push_oop(objAction);
	    args.push_oop(objCause);
	    args.push_oop(objGcInfo);
	
	    JavaCalls::call_virtual(&result,
	                            gc_mbean_klass,
	                            vmSymbols::createGCNotification_name(),
	                            vmSymbols::createGCNotification_signature(),
	                            &args,
	                            CHECK);
	    if (HAS_PENDING_EXCEPTION) {
	      CLEAR_PENDING_EXCEPTION;
	    }
	
  {- -------------------------------------------
  (1) 送信済みの通知オブジェクト(GCNotificationRequest)を削除する.
      ---------------------------------------- -}

	    delete request;
	  }
	}
	
```


