---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/frame.cpp

### 名前(function name)
```
void frame::ZapDeadClosure::do_oop(oop* p) {
```

### 本体部(body)
```
	  if (TraceZapDeadLocals) tty->print_cr("zapping @ " INTPTR_FORMAT " containing " INTPTR_FORMAT, p, (address)*p);
	  // Need cast because on _LP64 the conversion to oop is ambiguous.  Constant
	  // can be either long or int.
	  *p = (oop)(int)0xbabebabe;
	}
	
```


