---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp
### 説明(description)

```
// ----------------------------------------------------------------------------
// Parallel class loading check

```

### 名前(function name)
```
bool SystemDictionary::is_parallelCapable(Handle class_loader) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (UnsyncloadClass || class_loader.is_null()) return true;
	  if (AlwaysLockClassLoader) return false;
	  return java_lang_Class::parallelCapable(class_loader());
	}
	
```


