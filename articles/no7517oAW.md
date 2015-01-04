---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp

### 名前(function name)
```
void instanceKlass::link_class(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(is_loaded(), "must be loaded");

  {- -------------------------------------------
  (1) まだリンクされてなければ, 
      instanceKlass::link_class_impl() を呼び出してリンク処理を行う.
      ---------------------------------------- -}

	  if (!is_linked()) {
	    instanceKlassHandle this_oop(THREAD, this->as_klassOop());
	    link_class_impl(this_oop, true, CHECK);
	  }
	}
	
```


