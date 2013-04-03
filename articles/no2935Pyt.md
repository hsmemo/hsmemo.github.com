---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/fieldDescriptor.hpp

### 名前(function name)
```
  void set_is_field_access_watched(const bool value)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) value 引数の価に応じて, _access_flags の JVM_ACC_FIELD_ACCESS_WATCHED ビットを立てる(あるいはクリアする)だけ.
      ---------------------------------------- -}

	                                          { _access_flags.set_is_field_access_watched(value); }
	
```


