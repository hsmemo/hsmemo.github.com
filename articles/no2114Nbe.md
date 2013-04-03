---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/shared/markSweep.cpp

### 名前(function name)
```
void MarkSweep::adjust_marks() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert( _preserved_oop_stack.size() == _preserved_mark_stack.size(),
	         "inconsistent preserved oop stacks");
	
  {- -------------------------------------------
  (1) まず _preserved_marks の中に入っている全て oop について, 
      PreservedMark::adjust_pointer() でポインタの修正を行う.
      ---------------------------------------- -}

	  // adjust the oops we saved earlier
	  for (size_t i = 0; i < _preserved_count; i++) {
	    _preserved_marks[i].adjust_pointer();
	  }
	
  {- -------------------------------------------
  (1) 次に, _preserved_oop_stack 内の全ての oop について, 
      MarkSweep::adjust_pointer() でポインタの修正処理を行う.
      ---------------------------------------- -}

	  // deal with the overflow stack
	  StackIterator<oop> iter(_preserved_oop_stack);
	  while (!iter.is_empty()) {
	    oop* p = iter.next_addr();
	    adjust_pointer(p);
	  }
	}
	
```


