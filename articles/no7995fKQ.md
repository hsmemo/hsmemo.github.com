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
  static SharkNativeWrapper* build(SharkBuilder* builder,
                                   methodHandle  target,
                                   const char*   name,
                                   BasicType*    arg_types,
                                   BasicType     return_type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい SharkNativeWrapper オブジェクトを作ってリターンする.
      ---------------------------------------- -}

	    return new SharkNativeWrapper(builder,
	                                  target,
	                                  name,
	                                  arg_types,
	                                  return_type);
	  }
	
```


