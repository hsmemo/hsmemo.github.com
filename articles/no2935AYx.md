---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEventController.cpp

### 名前(function name)
```
void VM_EnterInterpOnlyMode::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JvmtiThreadState::invalidate_cur_stack_depth() を呼び出し, 
      JvmtiThreadState::_cur_stack_depth フィールドの値をリセットしておく.
    
      (これは FramePop イベントのための処理. (See: [here](no2935pZs.html) for details))
      ---------------------------------------- -}

	  // Set up the current stack depth for later tracking
	  _state->invalidate_cur_stack_depth();
	
  {- -------------------------------------------
  (1) JvmtiThreadState::enter_interp_only_mode() を呼んで
      対象の JavaThread 内の _interp_only_mode フィールドの値を increment しておく.
      ---------------------------------------- -}

	  _state->enter_interp_only_mode();
	
  {- -------------------------------------------
  (1) 対象のスレッドのスタック内に compiled フレームが有れば deopt する.
    
      (スタック内にある(ネイティブメソッドではない)メソッドのフレームを全てたどる.
      その中に JIT 生成されたメソッドのフレーム(= can_be_deoptimized() が true になるフレーム) があれば 
      対応する nmethod に CodeCache::mark_for_deoptimization() で印を付け,
      num_marked 変数もインクリメントしておく.
      スタックを辿り終わった後で num_marked が 0 以上であれば, VM_Deoptimize::doit() を呼び出す)
  
      (なお, VM_Deoptimize::doit() は 
       CodeCache::mark_for_deoptimization() で印を付けられたフレームを全て deopt する関数 (See: VM_Deoptimize))
      ---------------------------------------- -}

	  JavaThread *thread = _state->get_thread();
	  if (thread->has_last_Java_frame()) {
	    // If running in fullspeed mode, single stepping is implemented
	    // as follows: first, the interpreter does not dispatch to
	    // compiled code for threads that have single stepping enabled;
	    // second, we deoptimize all methods on the thread's stack when
	    // interpreted-only mode is enabled the first time for a given
	    // thread (nothing to do if no Java frames yet).
	    int num_marked = 0;
	    ResourceMark resMark;
	    RegisterMap rm(thread, false);
	    for (vframe* vf = thread->last_java_vframe(&rm); vf; vf = vf->sender()) {
	      if (can_be_deoptimized(vf)) {
	        ((compiledVFrame*) vf)->code()->mark_for_deoptimization();
	        ++num_marked;
	      }
	    }
	    if (num_marked > 0) {
	      VM_Deoptimize op;
	      VMThread::execute(&op);
	    }
	  }
	}
	
```


