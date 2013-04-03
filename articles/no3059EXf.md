---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) ネイティブコードの先頭アドレスをリターン
      ---------------------------------------- -}

	  address native_function() const                { return *(native_function_addr()); }
	
```


