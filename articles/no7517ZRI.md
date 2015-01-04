---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp

### 名前(function name)
```
klassOop SystemDictionary::resolve_or_null(Symbol* class_name, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数違いの SystemDictionary::resolve_or_null() を呼び出し, 結果をリターン.
      ---------------------------------------- -}

	  return resolve_or_null(class_name, Handle(), Handle(), THREAD);
	}
	
```


