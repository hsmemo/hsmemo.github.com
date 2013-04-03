---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/thread.cpp

### 名前(function name)
```
void JavaThread::remove_stack_guard_pages() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) guard page が設定されていない場合は
      (= _stack_guard_state フィールドが stack_guard_unused であれば), 
      することはないので, ここでリターン.
      ---------------------------------------- -}

	  if (_stack_guard_state == stack_guard_unused) return;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address low_addr = stack_base() - stack_size();
	  size_t len = (StackYellowPages + StackRedPages) * os::vm_page_size();
	
  {- -------------------------------------------
  (1) 以下の処理は, os::allocate_stack_guard_pages() の値に応じて 2通りに分岐.
      (なお, os::allocate_stack_guard_pages() は, 
       guard page 作成時にそのメモリ領域を新たにコミットしたかどうかを示す. true ならコミットした)
    
      * os::allocate_stack_guard_pages() が true の場合:
        os::remove_stack_guard_pages() で, 対象のメモリ領域をアンコミットする.
        成功したら, _stack_guard_state フィールドの値を stack_guard_unused にしておく.
        (失敗しても warning を出すだけ)
  
      * os::allocate_stack_guard_pages() が false の場合:
        os::unguard_memory() で, 対象領域のメモリ保護を解除する.
        成功したら, _stack_guard_state フィールドの値を stack_guard_unused にしておく.
        (失敗しても warning を出すだけ)
      ---------------------------------------- -}

	  if (os::allocate_stack_guard_pages()) {
	    if (os::remove_stack_guard_pages((char *) low_addr, len)) {
	      _stack_guard_state = stack_guard_unused;
	    } else {
	      warning("Attempt to deallocate stack guard pages failed.");
	    }
	  } else {
	    if (_stack_guard_state == stack_guard_unused) return;
	    if (os::unguard_memory((char *) low_addr, len)) {
	      _stack_guard_state = stack_guard_unused;
	    } else {
	        warning("Attempt to unprotect stack guard pages failed.");
	    }
	  }
	}
	
```


