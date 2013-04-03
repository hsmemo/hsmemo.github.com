---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/oopFactory.cpp

### 名前(function name)
```
objArrayOop oopFactory::new_objArray(klassOop klass, int length, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(klass->is_klass(), "must be instance class");

  {- -------------------------------------------
  (1) 以下のどちらかでメモリを確保し, 結果をリターン.
  
      * 処理対象のクラス(= klass 引数)が配列クラスの場合:
        arrayKlass::allocate_arrayArray()
      * 処理対象のクラス(= klass 引数)が配列クラスではない場合:
        instanceKlass::allocate_objArray() 
      ---------------------------------------- -}

	  if (klass->klass_part()->oop_is_array()) {
	    return ((arrayKlass*)klass->klass_part())->allocate_arrayArray(1, length, THREAD);
	  } else {
	    assert (klass->klass_part()->oop_is_instance(), "new object array with klass not an instanceKlass");
	    return ((instanceKlass*)klass->klass_part())->allocate_objArray(1, length, THREAD);
	  }
	}
	
```


