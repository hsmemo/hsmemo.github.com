---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/linkResolver.cpp

### 名前(function name)
```
void LinkResolver::resolve_field(FieldAccessInfo& result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数違いの LinkResolver::resolve_field() を呼び出すだけ.
      ---------------------------------------- -}

	  resolve_field(result, pool, index, byte, check_only, true, CHECK);
	}
	
```


