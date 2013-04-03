---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/sparsePRT.cpp

### 名前(function name)
```
bool SparsePRT::should_be_on_expanded_list() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  if (_expanded) {
	    assert(_cur != _next, "if _expanded is true, cur should be != _next");
	  } else {
	    assert(_cur == _next, "if _expanded is false, cur should be == _next");
	  }

  {- -------------------------------------------
  (1) この SparsePRT が拡張されていれば true を返す
      (というか, SparsePRT::expanded() の返値をそのままリターンするだけ).
      ---------------------------------------- -}

	  return expanded();
	}
	
```


