---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp

### 名前(function name)
```
int AbstractInterpreter::layout_activation(methodOop method,
                                           int tempcount,
                                           int popframe_extra_args,
                                           int moncount,
                                           int caller_actual_parameters,
                                           int callee_param_count,
                                           int callee_locals,
                                           frame* caller,
                                           frame* interpreter_frame,
                                           bool is_top_frame) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Note: This calculation must exactly parallel the frame setup
	  // in AbstractInterpreterGenerator::generate_method_entry.
	  // If interpreter_frame!=NULL, set up the method, locals, and monitors.
	  // The frame interpreter_frame, if not NULL, is guaranteed to be the
	  // right size, as determined by a previous call to this method.
	  // It is also guaranteed to be walkable even though it is in a skeletal state
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // fixed size of an interpreter frame:
	  int max_locals = method->max_locals() * Interpreter::stackElementWords;
	  int extra_locals = (method->max_locals() - method->size_of_parameters()) *
	                     Interpreter::stackElementWords;
	
  {- -------------------------------------------
  (1) フレームサイズを計算する (以下の局所変数 size に格納).
      ---------------------------------------- -}

	  int overhead = frame::sender_sp_offset -
	                 frame::interpreter_frame_initial_sp_offset;
	  // Our locals were accounted for by the caller (or last_frame_adjust
	  // on the transistion) Since the callee parameters already account
	  // for the callee's params we only need to account for the extra
	  // locals.
	  int size = overhead +
	         (callee_locals - callee_param_count)*Interpreter::stackElementWords +
	         moncount * frame::interpreter_frame_monitor_size() +
	         tempcount* Interpreter::stackElementWords + popframe_extra_args;

  {- -------------------------------------------
  (1) (interpreter_frame 引数が NULL でなければ, 以降の処理でレジスタ値の設定を行う)
      ---------------------------------------- -}

	  if (interpreter_frame != NULL) {

    {- -------------------------------------------
  (1.1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
        ---------------------------------------- -}

	#ifdef ASSERT
	    if (!EnableInvokeDynamic)
	      // @@@ FIXME: Should we correct interpreter_frame_sender_sp in the calling sequences?
	      // Probably, since deoptimization doesn't work yet.
	      assert(caller->unextended_sp() == interpreter_frame->interpreter_frame_sender_sp(), "Frame not properly walkable");
	    assert(caller->sp() == interpreter_frame->sender_sp(), "Frame not properly walkable(2)");
	#endif
	
    {- -------------------------------------------
  (1.1) 実行するメソッド(に対応する methodOop) を設定
        ---------------------------------------- -}

	    interpreter_frame->interpreter_frame_set_method(method);

    {- -------------------------------------------
  (1.1) 局所変数領域の先頭アドレス(?)を設定
        ---------------------------------------- -}

	    // NOTE the difference in using sender_sp and
	    // interpreter_frame_sender_sp interpreter_frame_sender_sp is
	    // the original sp of the caller (the unextended_sp) and
	    // sender_sp is fp+16 XXX
	    intptr_t* locals = interpreter_frame->sender_sp() + max_locals - 1;
	
    {- -------------------------------------------
  (1.1) interpreter_frame_monitor_block_top を設定
        ---------------------------------------- -}

	    interpreter_frame->interpreter_frame_set_locals(locals);
	    BasicObjectLock* montop = interpreter_frame->interpreter_frame_monitor_begin();
	    BasicObjectLock* monbot = montop - moncount;
	    interpreter_frame->interpreter_frame_set_monitor_end(monbot);
	
    {- -------------------------------------------
  (1.1) interpreter_frame_last_sp を設定
        ---------------------------------------- -}

	    // Set last_sp
	    intptr_t*  esp = (intptr_t*) monbot -
	                     tempcount*Interpreter::stackElementWords -
	                     popframe_extra_args;
	    interpreter_frame->interpreter_frame_set_last_sp(esp);
	
    {- -------------------------------------------
  (1.1) interpreter_frame_sender_sp を設定
        ---------------------------------------- -}

	    // All frames but the initial (oldest) interpreter frame we fill in have
	    // a value for sender_sp that allows walking the stack but isn't
	    // truly correct. Correct the value here.
	    if (extra_locals != 0 &&
	        interpreter_frame->sender_sp() ==
	        interpreter_frame->interpreter_frame_sender_sp()) {
	      interpreter_frame->set_interpreter_frame_sender_sp(caller->sp() +
	                                                         extra_locals);
	    }

    {- -------------------------------------------
  (1.1) interpreter_frame_cache を設定  (<= constantPoolCache)
        ---------------------------------------- -}

	    *interpreter_frame->interpreter_frame_cache_addr() =
	      method->constants()->cache();
	  }

  {- -------------------------------------------
  (1) 計算したフレームサイズをリターンする
      ---------------------------------------- -}

	  return size;
	}
	
```


