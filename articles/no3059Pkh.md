---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreterGenerator.hpp

### 名前(function name)
```
  address generate_exception_handler(const char* name, const char* message) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_exception_handler_common() を呼び出すだけ.
      ---------------------------------------- -}

	    return generate_exception_handler_common(name, message, false);
	  }
	
```


