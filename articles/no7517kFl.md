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
// Forwards to resolve_or_null

```

### 名前(function name)
```
klassOop SystemDictionary::resolve_or_fail(Symbol* class_name, Handle class_loader, Handle protection_domain, bool throw_error, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SystemDictionary::resolve_or_null() を呼んで, 指定されたクラスを取得する.
      (まだロードされていなければロードも行う)
      ---------------------------------------- -}

	  klassOop klass = resolve_or_null(class_name, class_loader, protection_domain, THREAD);

  {- -------------------------------------------
  (1) もし例外が発生していたり, あるいは取得結果が NULL だった場合は, 
      SystemDictionary::handle_resolution_exception() を呼んで, 
      適切な例外を発生させたり結果を調整したりする.(#TODO)
      ---------------------------------------- -}

	  if (HAS_PENDING_EXCEPTION || klass == NULL) {
	    KlassHandle k_h(THREAD, klass);
	    // can return a null klass
	    klass = handle_resolution_exception(class_name, class_loader, protection_domain, throw_error, k_h, THREAD);
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return klass;
	}
	
```


