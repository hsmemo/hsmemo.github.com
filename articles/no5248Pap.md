---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/serviceUtil.hpp
### 説明(description)

```
  // Return true if oop represents an object that is "visible"
  // to the java world.
```

### 名前(function name)
```
  static inline bool visible_oop(oop o) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) o 引数に応じて true または false をリターンする.
    
      * true をリターンするケース: 
        * instanceOop であり(= is_instance() が true), かつ
          クラスオブジェクトではない (= klass() が java.lang.Class ではない) 場合
        * プリミティブ型用のクラスオブジェクトである (= java_lang_Class::is_primitive() が true) 場合
        * 何らかの Java クラス用のクラスオブジェクトである場合 
        * 何らかの Java クラスの配列用のクラスオブジェクトである場合 
        * プリミティブ型の配列クラス用のクラスオブジェクトである場合 
        * objArrayOop であり(= is_objArray() が true), 
          かつ Universe::systemObjArrayKlassObj() ではない場合
        * typeArrayOop の場合 (= is_typeArray() が true)
  
      * false をリターンするケース:
        * 上記以外の全ての場合
    
        なお, 以下の2つのケースだけは明示的にチェックして false をリターンしている.
        * JNIHandles::deleted_handle() の場合
        * klassOop の場合 (= is_klass() が true)
  
      ---------------------------------------- -}

	    // the sentinel for deleted handles isn't visible
	    if (o == JNIHandles::deleted_handle()) {
	      return false;
	    }
	
	    // ignore KlassKlass
	    if (o->is_klass()) {
	      return false;
	    }
	
	    // instance
	    if (o->is_instance()) {
	      // instance objects are visible
	      if (o->klass() != SystemDictionary::Class_klass()) {
	        return true;
	      }
	      if (java_lang_Class::is_primitive(o)) {
	        return true;
	      }
	      // java.lang.Classes are visible
	      o = java_lang_Class::as_klassOop(o);
	      if (o->is_klass()) {
	        // if it's a class for an object, an object array, or
	        // primitive (type) array then it's visible.
	        klassOop klass = (klassOop)o;
	        if (Klass::cast(klass)->oop_is_instance()) {
	          return true;
	        }
	        if (Klass::cast(klass)->oop_is_objArray()) {
	          return true;
	        }
	        if (Klass::cast(klass)->oop_is_typeArray()) {
	          return true;
	        }
	      }
	      return false;
	    }
	    // object arrays are visible if they aren't system object arrays
	    if (o->is_objArray()) {
	      objArrayOop array = (objArrayOop)o;
	      if (array->klass() != Universe::systemObjArrayKlassObj()) {
	        return true;
	      } else {
	        return false;
	      }
	    }
	    // type arrays are visible
	    if (o->is_typeArray()) {
	      return true;
	    }
	    // everything else (methodOops, ...) aren't visible
	    return false;
	  };   // end of visible_oop()
	
```


