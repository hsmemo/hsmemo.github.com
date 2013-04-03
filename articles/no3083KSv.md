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
  (1) (単なる getter method)
      ---------------------------------------- -}

	  volatile address from_interpreted_entry() const{ return (address)OrderAccess::load_ptr_acquire(&_from_interpreted_entry); }
	
```


