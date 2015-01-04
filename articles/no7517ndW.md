---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp
### 説明(description)

```
// Forwards to resolve_instance_class_or_null

```

### 名前(function name)
```
klassOop SystemDictionary::resolve_array_class_or_null(Symbol* class_name,
                                                       Handle class_loader,
                                                       Handle protection_domain,
                                                       TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(FieldType::is_array(class_name), "must be array");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  klassOop k = NULL;
	  FieldArrayInfo fd;
	  // dimension and object_key in FieldArrayInfo are assigned as a side-effect
	  // of this call
	  BasicType t = FieldType::get_array_info(class_name, fd, CHECK_NULL);

  {- -------------------------------------------
  (1) 結果を取得する.
      取得は, 型に応じて以下のように行う.
      * オブジェクト型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
        SystemDictionary::resolve_instance_class_or_null() 及び Klass::array_klass() で取得.
      * プリミティブ型の配列クラスの場合: (一次元配列のクラスだけでなく, 多次元配列のクラスも含む)
        Universe::typeArrayKlassObj() 及び Klass::array_klass() で取得.
      ---------------------------------------- -}

	  if (t == T_OBJECT) {
	    // naked oop "k" is OK here -- we assign back into it
	    k = SystemDictionary::resolve_instance_class_or_null(fd.object_key(),
	                                                         class_loader,
	                                                         protection_domain,
	                                                         CHECK_NULL);
	    if (k != NULL) {
	      k = Klass::cast(k)->array_klass(fd.dimension(), CHECK_NULL);
	    }
	  } else {
	    k = Universe::typeArrayKlassObj(t);
	    k = typeArrayKlass::cast(k)->array_klass(fd.dimension(), CHECK_NULL);
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return k;
	}
	
```


