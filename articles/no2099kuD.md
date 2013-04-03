---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/verifier.cpp

### 名前(function name)
```
bool Verifier::should_verify_for(oop class_loader, bool should_verify_class) {
```

### 本体部(body)
```
	  return (class_loader == NULL || !should_verify_class) ?
	    BytecodeVerificationLocal : BytecodeVerificationRemote;
	}
	
```


