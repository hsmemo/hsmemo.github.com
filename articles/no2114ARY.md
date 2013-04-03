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
void MarkSweep::restore_marks() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_preserved_oop_stack.size() == _preserved_mark_stack.size(),
	         "inconsistent preserved oop stacks");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintGC && Verbose) {
	    gclog_or_tty->print_cr("Restoring %d marks",
	                           _preserved_count + _preserved_oop_stack.size());
	  }
	
  {- -------------------------------------------
  (1) まず _preserved_marks の中に入っている全て oop について, 
      PreservedMark::restore() で mark 値の復帰処理を行う.
      ---------------------------------------- -}

	  // restore the marks we saved earlier
	  for (size_t i = 0; i < _preserved_count; i++) {
	    _preserved_marks[i].restore();
	  }
	
  {- -------------------------------------------
  (1) 次に, _preserved_oop_stack 内の全ての oop について, 
      oopDesc::set_mark() で mark 値の復帰処理を行う.
      ---------------------------------------- -}

	  // deal with the overflow
	  while (!_preserved_oop_stack.is_empty()) {
	    oop obj       = _preserved_oop_stack.pop();
	    markOop mark  = _preserved_mark_stack.pop();
	    obj->set_mark(mark);
	  }
	}
	
```


