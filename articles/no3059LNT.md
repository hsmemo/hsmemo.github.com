---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/invocationCounter.cpp

### 名前(function name)
```
void InvocationCounter::def(State state, int init, Action action) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(0 <= state && state < number_of_states, "illegal state");
	  assert(0 <= init  && init  < count_limit, "initial value out of range");

  {- -------------------------------------------
  (1) フィールドの初期化
      (InvocationCounter クラスの static フィールドの初期化)
      ---------------------------------------- -}

	  _init  [state] = init;
	  _action[state] = action;
	}
	
```


