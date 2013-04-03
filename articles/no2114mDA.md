---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/oop.inline.hpp

### 名前(function name)
```
inline int oopDesc::adjust_pointers() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 対応する Klass オブジェクトの Klass::oop_adjust_pointers() を呼び出すだけ.
      ---------------------------------------- -}

	  debug_only(int check_size = size());
	  int s = blueprint()->oop_adjust_pointers(this);
	  assert(s == check_size, "should be the same");
	  return s;
	}
	
```


