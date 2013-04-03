---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp
### 説明(description)

```
// See if we've got enough room on the stack for locals plus overhead.
// The expression stack grows down incrementally, so the normal guard
// page mechanism will work for that.
//
// NOTE: Since the additional locals are also always pushed (wasn't
// obvious in generate_method_entry) so the guard should work for them
// too.
//
// Args:
//      rdx: number of additional locals this frame needs (what we must check)
//      rbx: methodOop
//
// Kills:
//      rax
```

### 名前(function name)
```
void InterpreterGenerator::generate_stack_overflow_check(void) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (page_size は, 実行環境におけるメモリページ 1 枚の大きさ)
      ---------------------------------------- -}

	  // monitor entry size: see picture of stack set
	  // (generate_method_entry) and frame_amd64.hpp
	  const int entry_size = frame::interpreter_frame_monitor_size() * wordSize;
	
	  // total overhead size: entry_size + (saved rbp through expr stack
	  // bottom).  be sure to change this if you add/subtract anything
	  // to/from the overhead area
	  const int overhead_size =
	    -(frame::interpreter_frame_initial_sp_offset * wordSize) + entry_size;
	
	  const int page_size = os::vm_page_size();
	
	  Label after_frame_check;
	
  {- -------------------------------------------
  (1) コード生成:
      「まずは, 引数で指定されたフレームサイズ(rdx)が, 1メモリページ(page_size)を超えるかどうかをチェック.
        (なお正確には, フレームサイズはモニター領域などによって少し増えるため, 
         その分も加味して超えるかどうかをチェックする.
         上の overhead_size でその追加分を計算している.).
  
        もしフレームサイズが 1ページ以下であれば, スタックオーバーフローの心配は無いので, 
        after_frame_check ラベルまでジャンプ. 」
      ---------------------------------------- -}

	  // see if the frame is greater than one page in size. If so,
	  // then we need to verify there is enough stack space remaining
	  // for the additional locals.
	  __ cmpl(rdx, (page_size - overhead_size) / Interpreter::stackElementSize);
	  __ jcc(Assembler::belowEqual, after_frame_check);
	
  {- -------------------------------------------
  (1) (以下で, 現在の RSP とプロテクションページの間に十分な空き領域があるかどうかを調べる)
      ---------------------------------------- -}

	  // compute rsp as if this were going to be the last frame on
	  // the stack before the red zone
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Address stack_base(r15_thread, Thread::stack_base_offset());
	  const Address stack_size(r15_thread, Thread::stack_size_offset());
	
  {- -------------------------------------------
  (1) コード生成:
      「要求されたフレームサイズに追加分を足し込んだ値を計算しておく (これが実際に確保されるフレームサイズ).
        結果は rax に格納」
      ---------------------------------------- -}

	  // locals + overhead, in bytes
	  __ mov(rax, rdx);
	  __ shlptr(rax, Interpreter::logStackElementSize);  // 2 slots per parameter.
	  __ addptr(rax, overhead_size);
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  Label stack_base_okay, stack_size_okay;
	  // verify that thread stack base is non-zero
	  __ cmpptr(stack_base, (int32_t)NULL_WORD);
	  __ jcc(Assembler::notEqual, stack_base_okay);
	  __ stop("stack base is zero");
	  __ bind(stack_base_okay);
	  // verify that thread stack size is non-zero
	  __ cmpptr(stack_size, 0);
	  __ jcc(Assembler::notEqual, stack_size_okay);
	  __ stop("stack size is zero");
	  __ bind(stack_size_okay);
	#endif
	
  {- -------------------------------------------
  (1) コード生成:
      「もし, yellow page 直前のアドレスと現在の SP の間が
        実際に確保するフレームサイズ分よりも広ければ, 
        スタックオーバーフローの心配は無いので after_frame_check ラベルまでジャンプ.」
    
       (なお, 実際に判定に用いているのは以下のような式.
          (stack_bottom - stack_length) + frame_size + StackRedPages + StackYellowPages > SP)
      ---------------------------------------- -}

	  // Add stack base to locals and subtract stack size
	  __ addptr(rax, stack_base);
	  __ subptr(rax, stack_size);
	
	  // Use the maximum number of pages we might bang.
	  const int max_pages = StackShadowPages > (StackRedPages+StackYellowPages) ? StackShadowPages :
	                                                                              (StackRedPages+StackYellowPages);
	
	  // add in the red and yellow zone sizes
	  __ addptr(rax, max_pages * page_size);
	
	  // check against the current stack bottom
	  __ cmpptr(rsp, rax);
	  __ jcc(Assembler::above, after_frame_check);
	
  {- -------------------------------------------
  (1) コード生成:
      「(十分な空き領域がないと分かったので) StackOverflowError を送出する.
  
        (なお送出する前に, 現在のリターンアドレスを rax レジスタに格納している)」
      ---------------------------------------- -}

	  __ pop(rax); // get return address
	  __ jump(ExternalAddress(Interpreter::throw_StackOverflowError_entry()));
	
  {- -------------------------------------------
  (1) コード生成:
      「(ここに到達したら, スタックには十分な空き領域があるということ)」
      ---------------------------------------- -}

	  // all done with frame size check
	  __ bind(after_frame_check);
	}
	
```


