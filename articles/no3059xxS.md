---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/bytecodes.cpp

### 名前(function name)
```
void Bytecodes::def(Code code, const char* name, const char* format, const char* wide_format, BasicType result_type, int depth, bool can_trap, Code java_code) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(wide_format == NULL || format != NULL, "short form must exist if there's a wide form");

  {- -------------------------------------------
  (1) フィールドの初期化 (_name, _result_type, _depth, _lengths, _java_code)
      ---------------------------------------- -}

	  int len  = (format      != NULL ? (int) strlen(format)      : 0);
	  int wlen = (wide_format != NULL ? (int) strlen(wide_format) : 0);
	  _name          [code] = name;
	  _result_type   [code] = result_type;
	  _depth         [code] = depth;
	  _lengths       [code] = (wlen << 4) | (len & 0xF);
	  _java_code     [code] = java_code;

  {- -------------------------------------------
  (1) Bytecodes::compute_flags() で format 文字列を解析し, 
      解析結果を _flags フィールドにセット.
      ---------------------------------------- -}

	  int bc_flags = 0;
	  if (can_trap)           bc_flags |= _bc_can_trap;
	  if (java_code != code)  bc_flags |= _bc_can_rewrite;
	  _flags[(u1)code+0*(1<<BitsPerByte)] = compute_flags(format,      bc_flags);
	  _flags[(u1)code+1*(1<<BitsPerByte)] = compute_flags(wide_format, bc_flags);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(is_defined(code)      == (format != NULL),      "");
	  assert(wide_is_defined(code) == (wide_format != NULL), "");
	  assert(length_for(code)      == len, "");
	  assert(wide_length_for(code) == wlen, "");
	}
	
```


