---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp
### 説明(description)

```
// If this is a growable mapping, remove the guard pages entirely by
// munmap()ping them.  If not, just call uncommit_memory(). This only
// affects the main/initial thread, but guard against future OS changes
```

### 名前(function name)
```
bool os::remove_stack_guard_pages(char* addr, size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  uintptr_t stack_extent, stack_base;
	  bool chk_bounds = NOT_DEBUG(os::Linux::is_initial_thread()) DEBUG_ONLY(true);

  {- -------------------------------------------
  (1) 以下のどちらかの処理を行う.
      * guard page を作成した時点で, その領域に既にメモリがマッピングされていた場合: 
        munmap() で guard page 領域のメモリマッピングを解除.
      * そうでない場合: 
        os::uncommit_memory() で物理メモリの割り当てだけを解除.
      ---------------------------------------- -}

	  if (chk_bounds && get_stack_bounds(&stack_extent, &stack_base)) {
	      assert(os::Linux::is_initial_thread(),
	           "growable stack in non-initial thread");
	
	    return ::munmap(addr, size) == 0;
	  }
	
	  return os::uncommit_memory(addr, size);
	}
	
```


