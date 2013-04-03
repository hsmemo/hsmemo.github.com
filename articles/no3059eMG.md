---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp
### 説明(description)

```
// Jump to from_interpreted entry of a call unless single stepping is possible
// in this thread in which case we must call the i2i entry
```

### 名前(function name)
```
void InterpreterMacroAssembler::jump_from_interpreted(Register method, Register temp) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し前の SP の値を待避しておく.」
      ---------------------------------------- -}

	  prepare_to_jump_from_interpreted();
	
  {- -------------------------------------------
  (1) コード生成: (ただし, JvmtiExport::can_post_interpreter_events() が false であれば
                 interp_only_mode のチェックは不要なので生成しない)
  
      「(JVMTI の状況によっては, 実行がインタープリタのみに制限されることがある. See: [here](no3059eFS.html) for details)
        
        カレントスレッドの interp_only_mode フィールドの値を確認する.
        もし true であれば, インタープリタ実行用のエントリポイントを使わないといけないため, 
        呼び出し対象の methodOop の interpreter_entry フィールドに入っているアドレスにジャンプする.
        (これがインタープリタ実行用のエントリポイント.
         JIT 対象になったとしてもこちらのエントリポイントは変更されない. See: ...#TODO)
      
        (true でなければ, このままフォールスルーして通常の呼び出し処理を行う)」
      ---------------------------------------- -}

	  if (JvmtiExport::can_post_interpreter_events()) {
	    Label run_compiled_code;
	    // JVMTI events, such as single-stepping, are implemented partly by avoiding running
	    // compiled code in threads for which the event is enabled.  Check here for
	    // interp_only_mode if these events CAN be enabled.
	    // interp_only is an int, on little endian it is sufficient to test the byte only
	    // Is a cmpl faster?
	    cmpb(Address(r15_thread, JavaThread::interp_only_mode_offset()), 0);
	    jcc(Assembler::zero, run_compiled_code);
	    jmp(Address(method, methodOopDesc::interpreter_entry_offset()));
	    bind(run_compiled_code);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し対象の methodOop の from_interpreted フィールドに入っているアドレスにジャンプする.
       (これが呼び出し対象のエントリポイント)」
      ---------------------------------------- -}

	  jmp(Address(method, methodOopDesc::from_interpreted_offset()));
	
	}
	
```


