---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/g1CollectorPolicy.cpp

### 名前(function name)
```
  void append_and_print_cr(const char* format, ...) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	    va_list ap;
	    va_start(ap, format);
	    vappend(format, ap);
	    va_end(ap);
	    gclog_or_tty->print_cr("%s", _buffer);
	    _cur = _indent_level * INDENT_CHARS;
	  }
	
```


