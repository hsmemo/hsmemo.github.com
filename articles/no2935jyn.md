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
Bytecodes::Code methodOopDesc::orig_bytecode_at(int bci) const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  BreakpointInfo* bp = instanceKlass::cast(method_holder())->breakpoints();
	  for (; bp != NULL; bp = bp->next()) {
	    if (bp->match(this, bci)) {
	      return bp->orig_bytecode();
	    }
	  }
	  ShouldNotReachHere();
	  return Bytecodes::_shouldnotreachhere;
	}
	
```


