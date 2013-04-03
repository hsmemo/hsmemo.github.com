---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiExtensions.cpp
### 説明(description)

```
// return the list of extension events

```

### 名前(function name)
```
jvmtiError JvmtiExtensions::get_events(JvmtiEnv* env,
                                       jint* extension_count_ptr,
                                       jvmtiExtensionEventInfo** extensions)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(_ext_events != NULL, "registration not done");
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (See: ResourceTracker)
      ---------------------------------------- -}

	  ResourceTracker rt(env);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  jvmtiExtensionEventInfo* ext_events;
	  jvmtiError err = rt.allocate(_ext_events->length() * sizeof(jvmtiExtensionEventInfo),
	                               (unsigned char**)&ext_events);
	  if (err != JVMTI_ERROR_NONE) {
	    return err;
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  for (int i=0; i<_ext_events->length(); i++ ) {
	    ext_events[i].extension_event_index = _ext_events->at(i)->extension_event_index;
	
	    char *id = _ext_events->at(i)->id;
	    err = rt.allocate(strlen(id)+1, (unsigned char**)&(ext_events[i].id));
	    if (err != JVMTI_ERROR_NONE) {
	      return err;
	    }
	    strcpy(ext_events[i].id, id);
	
	    char *desc = _ext_events->at(i)->short_description;
	    err = rt.allocate(strlen(desc)+1,
	                      (unsigned char**)&(ext_events[i].short_description));
	    if (err != JVMTI_ERROR_NONE) {
	      return err;
	    }
	    strcpy(ext_events[i].short_description, desc);
	
	    // params
	
	    jint param_count = _ext_events->at(i)->param_count;
	
	    ext_events[i].param_count = param_count;
	    if (param_count == 0) {
	      ext_events[i].params = NULL;
	    } else {
	      err = rt.allocate(param_count*sizeof(jvmtiParamInfo),
	                        (unsigned char**)&(ext_events[i].params));
	      if (err != JVMTI_ERROR_NONE) {
	        return err;
	      }
	      jvmtiParamInfo* src_params = _ext_events->at(i)->params;
	      jvmtiParamInfo* dst_params = ext_events[i].params;
	
	      for (int j=0; j<param_count; j++) {
	        err = rt.allocate(strlen(src_params[j].name)+1,
	                          (unsigned char**)&(dst_params[j].name));
	        if (err != JVMTI_ERROR_NONE) {
	          return err;
	        }
	        strcpy(dst_params[j].name, src_params[j].name);
	
	        dst_params[j].kind = src_params[j].kind;
	        dst_params[j].base_type = src_params[j].base_type;
	        dst_params[j].null_ok = src_params[j].null_ok;
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  *extension_count_ptr = _ext_events->length();
	  *extensions = ext_events;
	  return JVMTI_ERROR_NONE;
	}
	
```


