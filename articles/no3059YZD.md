---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/compiler/compileBroker.cpp
### 説明(description)

```
// Set _should_block.
// Call this from the VM, with Threads_lock held and a safepoint requested.
```

### 名前(function name)
```
void CompileBroker::set_should_block() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(Threads_lock->owner() == Thread::current(), "must have threads lock");
	  assert(SafepointSynchronize::is_at_safepoint(), "must be at a safepoint already");

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (PrintCompilation && (Verbose || WizardMode))
	    tty->print_cr("notifying compiler thread pool to block");
	#endif

  {- -------------------------------------------
  (1) _should_block フィールドを true にするだけ.
      ---------------------------------------- -}

	  _should_block = true;
	}
	
```


