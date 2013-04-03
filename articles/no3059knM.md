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
void Bytecodes::def(Code code, const char* name, const char* format, const char* wide_format, BasicType result_type, int depth, bool can_trap) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数違いの Bytecodes::def() を呼ぶだけ
      ---------------------------------------- -}

	  def(code, name, format, wide_format, result_type, depth, can_trap, code);
	}
	
```


