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
  (1) instanceKlass::get_jmethod_id() を呼び出して, 結果をリターンするだけ.
      ---------------------------------------- -}

	  // Get this method's jmethodID -- allocate if it doesn't exist
	  jmethodID jmethod_id()                            { methodHandle this_h(this);
	                                                      return instanceKlass::get_jmethod_id(method_holder(), this_h); }
	
```


