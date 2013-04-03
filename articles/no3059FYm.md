---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/signature.cpp

### 名前(function name)
```
int SignatureIterator::parse_type() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Note: This function could be simplified by using "return T_XXX_size;"
	  //       instead of the assignment and the break statements. However, it
	  //       seems that the product build for win32_i486 with MS VC++ 6.0 doesn't
	  //       work (stack underflow for some tests) - this seems to be a VC++ 6.0
	  //       compiler bug (was problem - gri 4/27/2000).
	  int size = -1;

  {- -------------------------------------------
  (1) (以下, シグネチャ文字列の文字に応じて分岐し, 
       型に応じた do_<type>() メソッドを呼び出す.
       
       ついでに, 読み取った分だけ _index を前に進めている 
       (このため, 次に parse するときは次の文字からになる).
       
       返値は, それぞれの型に対応した T_<type>_size 定数値.)
      ---------------------------------------- -}

	  switch(_signature->byte_at(_index)) {
	    case 'B': do_byte  (); if (_parameter_index < 0 ) _return_type = T_BYTE;
	              _index++; size = T_BYTE_size   ; break;
	    case 'C': do_char  (); if (_parameter_index < 0 ) _return_type = T_CHAR;
	              _index++; size = T_CHAR_size   ; break;
	    case 'D': do_double(); if (_parameter_index < 0 ) _return_type = T_DOUBLE;
	              _index++; size = T_DOUBLE_size ; break;
	    case 'F': do_float (); if (_parameter_index < 0 ) _return_type = T_FLOAT;
	              _index++; size = T_FLOAT_size  ; break;
	    case 'I': do_int   (); if (_parameter_index < 0 ) _return_type = T_INT;
	              _index++; size = T_INT_size    ; break;
	    case 'J': do_long  (); if (_parameter_index < 0 ) _return_type = T_LONG;
	              _index++; size = T_LONG_size   ; break;
	    case 'S': do_short (); if (_parameter_index < 0 ) _return_type = T_SHORT;
	              _index++; size = T_SHORT_size  ; break;
	    case 'Z': do_bool  (); if (_parameter_index < 0 ) _return_type = T_BOOLEAN;
	              _index++; size = T_BOOLEAN_size; break;
	    case 'V': do_void  (); if (_parameter_index < 0 ) _return_type = T_VOID;
	              _index++; size = T_VOID_size;  ; break;
	    case 'L':
	      { int begin = ++_index;
	        Symbol* sig = _signature;
	        while (sig->byte_at(_index++) != ';') ;
	        do_object(begin, _index);
	      }
	      if (_parameter_index < 0 ) _return_type = T_OBJECT;
	      size = T_OBJECT_size;
	      break;
	    case '[':
	      { int begin = ++_index;
	        skip_optional_size();
	        Symbol* sig = _signature;
	        while (sig->byte_at(_index) == '[') {
	          _index++;
	          skip_optional_size();
	        }
	        if (sig->byte_at(_index) == 'L') {
	          while (sig->byte_at(_index++) != ';') ;
	        } else {
	          _index++;
	        }
	        do_array(begin, _index);
	       if (_parameter_index < 0 ) _return_type = T_ARRAY;
	      }
	      size = T_ARRAY_size;
	      break;
	    default:
	      ShouldNotReachHere();
	      break;
	  }
	  assert(size >= 0, "size must be set");
	  return size;
	}
	
```


