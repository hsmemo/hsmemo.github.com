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
klassOop SystemDictionary::resolve_or_null(Symbol* class_name, Handle class_loader, Handle protection_domain, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(!THREAD->is_Compiler_thread(), "Can not load classes with the Compiler thread");

  {- -------------------------------------------
  (1) 結果を取得する.
      取得は, 型に応じて以下のように行う.
      * 配列クラスの場合:
        SystemDictionary::resolve_array_class_or_null() で取得する.
      * 配列クラスではない場合
        SystemDictionary::resolve_instance_class_or_null() で取得する.
        (なお, オブジェクト型のクラスの場合は, 型シグネチャの最初の'L'と最後の';'は除去して呼び出す)
        (<= これに引っかからないケース(最後の else のケース)はプリミティブ型のクラスの場合? #TODO)
      ---------------------------------------- -}

	  if (FieldType::is_array(class_name)) {
	    return resolve_array_class_or_null(class_name, class_loader, protection_domain, CHECK_NULL);
	  } else if (FieldType::is_obj(class_name)) {
	    ResourceMark rm(THREAD);
	    // Ignore wrapping L and ;.
	    TempNewSymbol name = SymbolTable::new_symbol(class_name->as_C_string() + 1,
	                                   class_name->utf8_length() - 2, CHECK_NULL);
	    return resolve_instance_class_or_null(name, class_loader, protection_domain, CHECK_NULL);
	  } else {
	    return resolve_instance_class_or_null(class_name, class_loader, protection_domain, CHECK_NULL);
	  }
	}
	
```


