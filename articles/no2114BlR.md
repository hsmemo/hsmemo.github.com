---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/classLoadingService.cpp

### 名前(function name)
```
void ClassLoadingService::notify_class_unloaded(instanceKlass* k) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_CLASSLOAD_PROBE(unloaded, k, false);

  {- -------------------------------------------
  (1) クラスのアンロード数を 1 インクリメントしておく.
      ---------------------------------------- -}

	  // Classes that can be unloaded must be non-shared
	  _classes_unloaded_count->inc();
	
  {- -------------------------------------------
  (1) UsePerfData オプションが指定されていれば, 
      ロードしたクラスのバイト数分だけ _classbytes_unloaded をインクリメントしておく.
      また, そのクラスに含まれる全てのメソッドのバイト数分だけ, 
      _class_methods_size を減らしておく.
      ---------------------------------------- -}

	  if (UsePerfData) {
	    // add the class size
	    size_t size = compute_class_size(k);
	    _classbytes_unloaded->inc(size);
	
	    // Compute method size & subtract from running total.
	    // We are called during phase 1 of mark sweep, so it's
	    // still ok to iterate through methodOops here.
	    objArrayOop methods = k->methods();
	    for (int i = 0; i < methods->length(); i++) {
	      _class_methods_size->inc(-methods->obj_at(i)->size());
	    }
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceClassUnloading) {
	    ResourceMark rm;
	    tty->print_cr("[Unloading class %s]", k->external_name());
	  }
	}
	
```


