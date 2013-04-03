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
void methodOopDesc::clear_breakpoint(int bci) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(bci >= 0, "");

  {- -------------------------------------------
  (1) clear_matches() 関数を呼び出し, bci 引数に該当する箇所をブレークポイントから元に戻す.
      ---------------------------------------- -}

	  clear_matches(this, bci);
	}
	
```


