---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/universe.cpp

### 名前(function name)
```
  virtual void do_object(oop obj) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oop 引数がクラスオブジェクトであれば, 
      java_lang_Class::fixup_mirror() を呼んで mirror を生成する.
    
      (なお, クラスオブジェクトでない場合は何もしない)
      ---------------------------------------- -}

	    if (obj->is_klass()) {
	      EXCEPTION_MARK;
	      KlassHandle k(THREAD, klassOop(obj));
	      // We will never reach the CATCH below since Exceptions::_throw will cause
	      // the VM to exit if an exception is thrown during initialization
	      java_lang_Class::fixup_mirror(k, CATCH);
	      // This call unconditionally creates a new mirror for k,
	      // and links in k's component_mirror field if k is an array.
	      // If k is an objArray, k's element type must already have
	      // a mirror.  In other words, this closure must process
	      // the component type of an objArray k before it processes k.
	      // This works because the permgen iterator presents arrays
	      // and their component types in order of creation.
	    }
	  }
	
```


