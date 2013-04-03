---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp

### 名前(function name)
```
void Management::initialize(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ServiceThread を起動させる.
      ---------------------------------------- -}

	  // Start the service thread
	  ServiceThread::initialize();
	
  {- -------------------------------------------
  (1) ManagementServer オプション (-Dcom.sun.management オプション) が指定されていれば, 
      sun.management.Agent.startAgent() を呼び出して
      JMX Management Server を立ち上げる.
      ---------------------------------------- -}

	  if (ManagementServer) {
	    ResourceMark rm(THREAD);
	    HandleMark hm(THREAD);
	
	    // Load and initialize the sun.management.Agent class
	    // invoke startAgent method to start the management server
	    Handle loader = Handle(THREAD, SystemDictionary::java_system_loader());
	    klassOop k = SystemDictionary::resolve_or_fail(vmSymbols::sun_management_Agent(),
	                                                   loader,
	                                                   Handle(),
	                                                   true,
	                                                   CHECK);
	    instanceKlassHandle ik (THREAD, k);
	
	    JavaValue result(T_VOID);
	    JavaCalls::call_static(&result,
	                           ik,
	                           vmSymbols::startAgent_name(),
	                           vmSymbols::void_method_signature(),
	                           CHECK);
	  }
	}
	
```


