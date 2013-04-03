---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/iterator.hpp

### 名前(function name)
```
  virtual const bool should_remember_klasses() const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: RememberKlassesChecker)
      ---------------------------------------- -}

	    assert(!must_remember_klasses(), "Should have overriden this method.");

  {- -------------------------------------------
  (1) false をリターンするだけ.
      ---------------------------------------- -}

	    return false;
	  }
	
```


