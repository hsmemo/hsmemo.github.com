---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnv.cpp
### 説明(description)

```
// capabilities_ptr - pre-checked for NULL
```

### 名前(function name)
```
jvmtiError
JvmtiEnv::AddCapabilities(const jvmtiCapabilities* capabilities_ptr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiManageCapabilities::add_capabilities() を呼び出すだけ.
      結果は第4引数 (= get_capabilities()) に書き戻される.
  
      (なお, get_capabilities() と get_prohibited_capabilities() は
       それぞれ _current_capabilities 及び _prohibited_capabilities フィールドを指すポインタ.
       (See: JvmtiEnvBase::get_capabilities(), JvmtiEnvBase::get_prohibited_capabilities()))
      ---------------------------------------- -}

	  return JvmtiManageCapabilities::add_capabilities(get_capabilities(),
	                                                   get_prohibited_capabilities(),
	                                                   capabilities_ptr,
	                                                   get_capabilities());
	} /* end AddCapabilities */
	
```


