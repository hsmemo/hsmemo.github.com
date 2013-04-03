---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/heapRegion.hpp
### 説明(description)

```
  // Note the start of a marking phase. Record the
  // start of the unmarked area of the region here.
```

### 名前(function name)
```
  void note_start_of_marking(bool during_initial_mark) {
```

### 本体部(body)
```
	    init_top_at_conc_mark_count();
	    _next_marked_bytes = 0;
	    if (during_initial_mark && is_young() && !is_survivor())
	      _next_top_at_mark_start = bottom();
	    else
	      _next_top_at_mark_start = top();
	  }
	
```


