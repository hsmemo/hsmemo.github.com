---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/compilationPolicy.cpp
### 説明(description)

```
// Called at the end of the safepoint
```

### 名前(function name)
```
void NonTieredCompPolicy::do_safepoint_work() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) CounterDecay::decay() を呼んで, 
      InvocationCounter の値を半減させる処理を行う.
  
      (ただし, UseCounterDecay オプションが false にセットされている場合はこの処理は行わない)
      (また, まだ処理する必要が無ければ (= CounterDecay::is_decay_needed() が false であれば), この処理は行わない)
    
      (See: CounterDecay)
      ---------------------------------------- -}

	  if(UseCounterDecay && CounterDecay::is_decay_needed()) {
	    CounterDecay::decay();
	  }
	}
	
```


