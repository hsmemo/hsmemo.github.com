---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interpreterRT_sparc.cpp

### 名前(function name)
```
  virtual void pass_int() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) インタープリタのフレーム内に置かれている引数を
      native の ABI での適切な位置にコピーする.
      ---------------------------------------- -}

	    *_to++ = *(jint *)(_from+Interpreter::local_offset_in_bytes(0));
	    _from -= Interpreter::stackElementSize;

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    add_signature( non_float );
	  }
	
```


