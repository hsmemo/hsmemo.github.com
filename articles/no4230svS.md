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
void MacroAssembler::reset_last_Java_frame(bool clear_fp,
                                           bool clear_pc) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「カレントスレッドの last_Java_sp フィールドをクリアする.」
  
      (なお, last_Java_sp のクリアを最初に行う.
       これは, last_Java_sp に値が書き込まれている状態では他のフィールドにも値が入っている, と保証するため)
      ---------------------------------------- -}

	  // we must set sp to zero to clear frame
	  movptr(Address(r15_thread, JavaThread::last_Java_sp_offset()), NULL_WORD);

  {- -------------------------------------------
  (1) コード生成: (ただし, clear_fp 引数が true の場合にのみ生成)
      「カレントスレッドの last_Java_fp フィールドをクリアする.」
      ---------------------------------------- -}

	  // must clear fp, so that compiled frames are not confused; it is
	  // possible that we need it only for debugging
	  if (clear_fp) {
	    movptr(Address(r15_thread, JavaThread::last_Java_fp_offset()), NULL_WORD);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, clear_pc 引数が true の場合にのみ生成)
      「カレントスレッドの last_Java_pc フィールドをクリアする.」
      ---------------------------------------- -}

	  if (clear_pc) {
	    movptr(Address(r15_thread, JavaThread::last_Java_pc_offset()), NULL_WORD);
	  }
	}
	
```


