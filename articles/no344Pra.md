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
objArrayOop instanceKlass::allocate_objArray(int n, int length, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし指定された配列長(= length)が負値であれば, java_lang_NegativeArraySizeException.
      ---------------------------------------- -}

	  if (length < 0) THROW_0(vmSymbols::java_lang_NegativeArraySizeException());

  {- -------------------------------------------
  (1) もし指定された配列長が最大長(= typeArrayKlass::max_length())を超えていれば, OutOfMemoryError.
      ---------------------------------------- -}

	  if (length > arrayOopDesc::max_array_length(T_OBJECT)) {
	    report_java_out_of_memory("Requested array size exceeds VM limit");
	    THROW_OOP_0(Universe::out_of_memory_error_array_size());
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int size = objArrayOopDesc::object_size(length);
	  klassOop ak = array_klass(n, CHECK_NULL);
	  KlassHandle h_ak (THREAD, ak);

  {- -------------------------------------------
  (1) CollectedHeap::array_allocate() でメモリを確保し, リターンする.
      ---------------------------------------- -}

	  objArrayOop o =
	    (objArrayOop)CollectedHeap::array_allocate(h_ak, size, length, CHECK_NULL);
	  return o;
	}
	
```


