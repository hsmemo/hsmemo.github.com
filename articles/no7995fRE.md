---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/shark/sharkNativeWrapper.hpp

### 名前(function name)
```
  SharkNativeWrapper(SharkBuilder* builder,
                     methodHandle  target,
                     const char*   name,
                     BasicType*    arg_types,
                     BasicType     return_type)
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) フィールドの初期化を行った後, 
      SharkNativeWrapper::initialize() を呼び出す.
      ---------------------------------------- -}

	    : SharkCompileInvariants(NULL, builder),
	      _target(target),
	      _arg_types(arg_types),
	      _return_type(return_type),
	      _lock_slot_offset(0) { initialize(name); }
	
```


