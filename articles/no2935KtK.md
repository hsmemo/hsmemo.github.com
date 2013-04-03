---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp

### 名前(function name)
```
bool VM_RedefineClasses::is_modifiable_class(oop klass_mirror) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) klass_mirror 引数で指定されたオブジェクトが
      以下のどれかに当てはまれば false をリターンする. 
      当てはまらなければ true をリターンする.
    
      * プリミティブ型のクラスオブジェクトである (= java_lang_Class::is_primitive() が true)
      * クラスオブジェクトではない (= java_lang_Class::as_klassOop() が NULL)
      * 配列クラス用のクラスオブジェクトである (= oop_is_instance() が false)
      ---------------------------------------- -}

	  // classes for primitives cannot be redefined
	  if (java_lang_Class::is_primitive(klass_mirror)) {
	    return false;
	  }
	  klassOop the_class_oop = java_lang_Class::as_klassOop(klass_mirror);
	  // classes for arrays cannot be redefined
	  if (the_class_oop == NULL || !Klass::cast(the_class_oop)->oop_is_instance()) {
	    return false;
	  }
	  return true;
	}
	
```


