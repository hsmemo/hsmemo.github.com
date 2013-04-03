---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp
### 説明(description)

```
// update the access_flags for the field in the klass
```

### 名前(function name)
```
void
JvmtiEnvBase::update_klass_field_access_flag(fieldDescriptor *fd) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) fd 引数で指定された fieldDescriptor オブジェクトの _access_flags 情報を, 
      対応する instanceKlass の fields の中にコピーする
      ---------------------------------------- -}

	  instanceKlass* ik = instanceKlass::cast(fd->field_holder());
	  typeArrayOop fields = ik->fields();
	  fields->ushort_at_put(fd->index(), (jushort)fd->access_flags().as_short());
	}
	
```


