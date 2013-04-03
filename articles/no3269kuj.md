---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/javaClasses.cpp

### 名前(function name)
```
void java_lang_Class::fixup_mirror(KlassHandle k, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(instanceMirrorKlass::offset_of_static_fields() != 0, "must have been computed already");
	
  {- -------------------------------------------
  (1) static フィールドのオフセット情報を修正しておく.
      ---------------------------------------- -}

	  if (k->oop_is_instance()) {
	    // Fixup the offsets
	    instanceKlass::cast(k())->do_local_static_fields(&fixup_static_field, CHECK);
	  }

  {- -------------------------------------------
  (1) java_lang_Class::create_mirror() を呼んで mirror を生成する.
      ---------------------------------------- -}

	  create_mirror(k, CHECK);
	}
	
```


