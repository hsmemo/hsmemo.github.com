---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiThreadState.cpp

### 名前(function name)
```
void JvmtiThreadState::invalidate_cur_stack_depth() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Thread *cur = Thread::current();
	  uint32_t debug_bits = 0;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // The caller can be the VMThread at a safepoint, the current thread
	  // or the target thread must be suspended.
	  guarantee((cur->is_VM_thread() && SafepointSynchronize::is_at_safepoint()) ||
	    (JavaThread *)cur == get_thread() ||
	    JvmtiEnv::is_thread_fully_suspended(get_thread(), false, &debug_bits),
	    "sanity check");
	
  {- -------------------------------------------
  (1) _cur_stack_depth フィールドを UNKNOWN_STACK_DEPTH にしておくだけ.
  
      (_cur_stack_depth フィールドは FramePop の処理に使用されるフィールド.
       UNKNOWN_STACK_DEPTH にしておくと, 次回の JvmtiThreadState::cur_stack_depth() の呼び出し時に
       きちんとした数値が計算し直される. 
       逆に UNKNOWN_STACK_DEPTH になっていないと, メソッドの enter/exit 時に増減されるだけ. (See: [here](no2935pZs.html) for details))
      ---------------------------------------- -}

	  _cur_stack_depth = UNKNOWN_STACK_DEPTH;
	}
	
```


