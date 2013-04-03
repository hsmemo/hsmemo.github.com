---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/g1/concurrentG1RefineThread.cpp

### 名前(function name)
```
void ConcurrentG1RefineThread::sample_young_list_rs_lengths() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  G1CollectedHeap* g1h = G1CollectedHeap::heap();
	  G1CollectorPolicy* g1p = g1h->g1_policy();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (g1p->adaptive_young_list_length()) {
	    int regions_visited = 0;
	    g1h->young_list()->rs_length_sampling_init();
	    while (g1h->young_list()->rs_length_sampling_more()) {
	      g1h->young_list()->rs_length_sampling_next();
	      ++regions_visited;
	
	      // we try to yield every time we visit 10 regions
	      if (regions_visited == 10) {
	        if (_sts.should_yield()) {
	          _sts.yield("G1 refine");
	          // we just abandon the iteration
	          break;
	        }
	        regions_visited = 0;
	      }
	    }
	
	    g1p->check_prediction_validity();
	  }
	}
	
```


