---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp

### 名前(function name)
```
void MacroAssembler::set_last_Java_frame(Register last_java_sp,
                                         Register last_java_fp,
                                         address  last_java_pc) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (引数で last_java_sp に使うレジスタが指定されていなければ, SP を使うこととする)
      ---------------------------------------- -}

	  // determine last_java_sp register
	  if (!last_java_sp->is_valid()) {
	    last_java_sp = rsp;
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 引数で last_java_fp が指定されていなければ生成しない)
      「カレントスレッドの last_Java_fp フィールドに, 指定されたレジスタの値をセットする.」
      ---------------------------------------- -}

	  // last_java_fp is optional
	  if (last_java_fp->is_valid()) {
	    movptr(Address(r15_thread, JavaThread::last_Java_fp_offset()),
	           last_java_fp);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 引数で last_java_pc が指定されていなければ生成しない)
      「カレントスレッドの last_Java_pc フィールドに, 指定されたレジスタの値をセットする.」
      ---------------------------------------- -}

	  // last_java_pc is optional
	  if (last_java_pc != NULL) {
	    Address java_pc(r15_thread,
	                    JavaThread::frame_anchor_offset() + JavaFrameAnchor::last_Java_pc_offset());
	    lea(rscratch1, InternalAddress(last_java_pc));
	    movptr(java_pc, rscratch1);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの last_Java_sp フィールドをセットする.」
  
      (なお, last_Java_sp への書き込みは最後に行う.
       これは, last_Java_sp に値が書き込まれている状態では他のフィールドにも値が入っている, と保証するため)
      ---------------------------------------- -}

	  movptr(Address(r15_thread, JavaThread::last_Java_sp_offset()), last_java_sp);
	
```


