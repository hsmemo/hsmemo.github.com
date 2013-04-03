---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/biasedLocking.cpp

### 名前(function name)
```
static void enable_biased_locking(klassOop k) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 引数で渡された klassOop の prototype header を biased locking pattern に置き換える.
      ---------------------------------------- -}

	  Klass::cast(k)->set_prototype_header(markOopDesc::biased_locking_prototype());
	}
	
```


