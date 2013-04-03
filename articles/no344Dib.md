---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/adaptiveSizePolicy.hpp

### 名前(function name)
```
  bool print_test(uint count) {
```

### 本体部(body)
```
	    // A count of zero is a special value that indicates that the
	    // interval test should be ignored.  An interval is of zero is
	    // a special value that indicates that the interval test should
	    // always fail (never do the print based on the interval test).
	    return PrintGCDetails &&
	           UseAdaptiveSizePolicy &&
	           (UseParallelGC || UseConcMarkSweepGC) &&
	           (AdaptiveSizePolicyOutputInterval > 0) &&
	           ((count == 0) ||
	             ((count % AdaptiveSizePolicyOutputInterval) == 0));
	  }
	
```


