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
void JavaThread::create_stack_guard_pages() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下のどちらかが成り立つ場合は, 処理の必要が無いので, ここでリターン.
      * guard page を使わない場合 (= os::uses_stack_guard_pages() が false の場合)
      * 既に guard page を設定済みの場合 (= _stack_guard_state フィールドが stack_guard_unused ではない場合)
      ---------------------------------------- -}

	  if (! os::uses_stack_guard_pages() || _stack_guard_state != stack_guard_unused) return;

  {- -------------------------------------------
  (1) (変数宣言など)
      (allocate は, stack guard pages 分のメモリ空間を確保する必要があるかどうかを示す. true なら必要がある)
      ---------------------------------------- -}

	  address low_addr = stack_base() - stack_size();
	  size_t len = (StackYellowPages + StackRedPages) * os::vm_page_size();
	
	  int allocate = os::allocate_stack_guard_pages();
	  // warning("Guarding at " PTR_FORMAT " for len " SIZE_FORMAT "\n", low_addr, len);
	
  {- -------------------------------------------
  (1) メモリ空間の確保が必要な場合には, os::create_stack_guard_pages() で確保しておく.
      (なお, os::create_stack_guard_pages() が失敗した場合には, ここでリターン)
      ---------------------------------------- -}

	  if (allocate && !os::create_stack_guard_pages((char *) low_addr, len)) {
	    warning("Attempt to allocate stack guard pages failed.");
	    return;
	  }
	
  {- -------------------------------------------
  (1) os::guard_memory() を呼んで, guard page 領域をアクセス不可にしておく.
      os::guard_memory() が成功すれば, _stack_guard_state フィールドの値を stack_guard_enabled にしておく.
      (逆に, os::guard_memory() が失敗したら, 
       os::uncommit_memory() を呼んで guard page 領域のメモリマッピングを解除しておく)
      ---------------------------------------- -}

	  if (os::guard_memory((char *) low_addr, len)) {
	    _stack_guard_state = stack_guard_enabled;
	  } else {
	    warning("Attempt to protect stack guard pages failed.");
	    if (os::uncommit_memory((char *) low_addr, len)) {
	      warning("Attempt to deallocate stack guard pages failed.");
	    }
	  }
	}
	
```


