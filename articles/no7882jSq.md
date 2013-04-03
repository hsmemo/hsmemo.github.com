---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/sharedHeap.cpp

### 名前(function name)
```
void SharedHeap::change_strong_roots_parity() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // Also set the new collection parity.
	  assert(_strong_roots_parity >= 0 && _strong_roots_parity <= 2,
	         "Not in range.");

  {- -------------------------------------------
  (1) _strong_roots_parity の値を 1 つインクリメントする. 
      ただし, それによって 3 に達したら 1 に戻す.
      ---------------------------------------- -}

	  _strong_roots_parity++;
	  if (_strong_roots_parity == 3) _strong_roots_parity = 1;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_strong_roots_parity >= 1 && _strong_roots_parity <= 2,
	         "Not in range.");
	}
	
```


