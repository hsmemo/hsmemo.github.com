---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/typeArrayKlass.cpp

### 名前(function name)
```
typeArrayOop typeArrayKlass::allocate(int length, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(log2_element_size() >= 0, "bad scale");

  {- -------------------------------------------
  (1) もし指定された配列長(上記の length)が 0 以上の値で, かつ
      可能な最大長以下であれば(= length が typeArrayKlass::max_length() 以下であれば), 
      CollectedHeap::array_allocate() または CollectedHeap::large_typearray_allocate() でメモリを確保してリターンする.
        (確保サイズが CollectedHeap::large_typearray_limit() を越えていれば 
         CollectedHeap::large_typearray_allocate() を使用し, 
         そうでなければ CollectedHeap::array_allocate() を使用する.)
  
      (なお, typeArrayKlass::max_length() の値は arrayOopDesc::max_array_length() で計算されている模様.
       See: arrayOopDesc::max_array_length())
      ---------------------------------------- -}

	  if (length >= 0) {
	    if (length <= max_length()) {
	      size_t size = typeArrayOopDesc::object_size(layout_helper(), length);
	      KlassHandle h_k(THREAD, as_klassOop());
	      typeArrayOop t;
	      CollectedHeap* ch = Universe::heap();
	      if (size < ch->large_typearray_limit()) {
	        t = (typeArrayOop)CollectedHeap::array_allocate(h_k, (int)size, length, CHECK_NULL);
	      } else {
	        t = (typeArrayOop)CollectedHeap::large_typearray_allocate(h_k, (int)size, length, CHECK_NULL);
	      }
	      assert(t->is_parsable(), "Don't publish unless parsable");
	      return t;

  {- -------------------------------------------
  (1) もし指定された配列長が 0 以上だが最大長を超えていれば, OutOfMemoryError.
      ---------------------------------------- -}

	    } else {
	      report_java_out_of_memory("Requested array size exceeds VM limit");
	      THROW_OOP_0(Universe::out_of_memory_error_array_size());
	    }

  {- -------------------------------------------
  (1) もし指定された配列長が負値であれば, java_lang_NegativeArraySizeException.
      ---------------------------------------- -}

	  } else {
	    THROW_0(vmSymbols::java_lang_NegativeArraySizeException());
	  }
	}
	
```


