---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/os.cpp
### 説明(description)
// Returns true if the current stack pointer is above the stack shadow
// pages, false otherwise.



### 名前(function name)
```
bool os::stack_shadow_pages_available(Thread *thread, methodHandle method) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(StackRedPages > 0 && StackYellowPages > 0,"Sanity check");

  {- -------------------------------------------
  (1) yellow page 直前のアドレスにさらに shadow page 分を足したアドレスを求めて (以下の stack_limit + reserved_area), 
      os::current_stack_pointer() で取得した現在のスタックポインタの値 (以下の sp) と比較する.
      sp の方がまだ大きければ (shadow page 分の空きスペースが残っているということなので) true をリターン.
      そうでなければ false をリターン.
      ---------------------------------------- -}

	  address sp = current_stack_pointer();
	  // Check if we have StackShadowPages above the yellow zone.  This parameter
	  // is dependent on the depth of the maximum VM call stack possible from
	  // the handler for stack overflow.  'instanceof' in the stack overflow
	  // handler or a println uses at least 8k stack of VM and native code
	  // respectively.
	  const int framesize_in_bytes =
	    Interpreter::size_top_interpreter_activation(method()) * wordSize;
	  int reserved_area = ((StackShadowPages + StackRedPages + StackYellowPages)
	                      * vm_page_size()) + framesize_in_bytes;
	  // The very lower end of the stack
	  address stack_limit = thread->stack_base() - thread->stack_size();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  return (sp > (stack_limit + reserved_area));
	}
	
```


