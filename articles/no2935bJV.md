---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp

### 名前(function name)
```
jvmtiError
JvmtiEnv::SetFieldAccessWatch(fieldDescriptor* fdesc_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 同じフィールドに既にaccess watchが設定されていたら, ここでリターン(JVMTI_ERROR_NOT_FOUND).
      ---------------------------------------- -}

	  // make sure we haven't set this watch before
	  if (fdesc_ptr->is_field_access_watched()) return JVMTI_ERROR_DUPLICATE;

  {- -------------------------------------------
  (1) 引数で渡された fieldDescriptor オブジェクトの _access_flags のビットを立てる.
      ---------------------------------------- -}

	  fdesc_ptr->set_is_field_access_watched(true);

  {- -------------------------------------------
  (1) JvmtiEnvBase::update_klass_field_access_flag() を呼んで, 
      対応する instanceKlass の fields 箇所を変更しておく.
      ---------------------------------------- -}

	  update_klass_field_access_flag(fdesc_ptr);
	
  {- -------------------------------------------
  (1) JvmtiEventController::change_field_watch() を呼んで, 
      指定されたフィールドを watch point に登録する.
      ---------------------------------------- -}

	  JvmtiEventController::change_field_watch(JVMTI_EVENT_FIELD_ACCESS, true);
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end SetFieldAccessWatch */
	
```


