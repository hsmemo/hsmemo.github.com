---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::call_VM_leaf_base(Register thread_cache, address entry_point, int number_of_arguments) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_not_delayed();

  {- -------------------------------------------
  (1) コード生成:
      「call 命令で, 引数で指定されたエントリポイントを呼び出す.
        (第一引数として, カレントスレッドを表す JavaThread を渡す)」
  
      (なお, 呼び出しの前後で G2_thread の退避/復帰を行っている. 
       See: MacroAssembler::save_thread(), MacroAssembler::restore_thread())
      ---------------------------------------- -}

	  save_thread(thread_cache);
	  // do the call
	  call(entry_point, relocInfo::runtime_call_type);
	  delayed()->nop();
	  restore_thread(thread_cache);

  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      コード生成: (デバッグ用の処理)
      ---------------------------------------- -}

	#ifdef ASSERT
	  set(badHeapWordVal, G3);
	  set(badHeapWordVal, G4);
	  set(badHeapWordVal, G5);
	#endif
	}
	
```


