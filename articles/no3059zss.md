---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.cpp

### 名前(function name)
```
address TemplateInterpreter::return_entry(TosState state, int length) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(0 <= length && length < Interpreter::number_of_return_entries, "illegal length");

  {- -------------------------------------------
  (1) _return_entry 内の該当するエントリをリターンするだけ.
      ---------------------------------------- -}

	  return _return_entry[length].entry(state);
	}
	
```


