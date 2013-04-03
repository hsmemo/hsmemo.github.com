---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.cpp

### 名前(function name)
```
void methodOopDesc::set_signature_handler(address handler) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) signature handler スロットに引数で指定された値をセット.
      ---------------------------------------- -}

	  address* signature_handler =  signature_handler_addr();
	  *signature_handler = handler;
	}
	
```


