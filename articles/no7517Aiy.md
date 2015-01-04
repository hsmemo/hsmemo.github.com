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
klassOop SystemDictionary::resolve_or_fail(Symbol* class_name,
                                           bool throw_error, TRAPS)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数違いの SystemDictionary::resolve_or_fail() を呼び出し, 結果をリターン.
      ---------------------------------------- -}

	  return resolve_or_fail(class_name, Handle(), Handle(), throw_error, THREAD);
	}
	
```


