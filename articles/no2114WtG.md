---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/sun/management/GarbageCollectorImpl.c

### 名前(function name)
```
JNIEXPORT void JNICALL Java_sun_management_GarbageCollectorImpl_setNotificationEnabled
(JNIEnv *env, jobject dummy, jobject gc,jboolean enabled) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_SetGCNotificationEnabled() を呼んで, 
      対応する GCMemoryManager の _notification_enabled フィールドを変更するだけ.
  
      (なお, 引数で渡された GarbageCollectorImpl オブジェクトが NULL だった場合は NullPointerException)
      (また, jmm のバージョンが 1.2 以上でなければ何もしない)
      ---------------------------------------- -}
	
	    if (gc == NULL) {
	        JNU_ThrowNullPointerException(env, "Invalid GarbageCollectorMBean");
	        return;
	    }
	    if((jmm_version > JMM_VERSION_1_2)
	       || (jmm_version == JMM_VERSION_1_2 && ((jmm_version&0xFF)>=1))) {
	      jmm_interface->SetGCNotificationEnabled(env, gc, enabled);
	    }
	}
	
```


