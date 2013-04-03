---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1RemSet.cpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) TriggerClosure::do_oop_nv() を呼び出すだけ.
      ---------------------------------------- -}

	  virtual void do_oop(oop* p)        { do_oop_nv(p); }
	  virtual void do_oop(narrowOop* p)  { do_oop_nv(p); }
	
```


