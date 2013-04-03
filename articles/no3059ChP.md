---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::throw_ArrayIndexOutOfBoundsException(JavaThread* thread, char* name, jint index))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  char message[jintAsStringSize];

  {- -------------------------------------------
  (1) (変数宣言など)
      (s は, 引数で指定された名前の例外クラスを SymbolTable 内から lookup したもの)
      ---------------------------------------- -}

	  // lookup exception klass
	  TempNewSymbol s = SymbolTable::new_symbol(name, CHECK);

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  if (ProfileTraps) {
	    note_trap(thread, Deoptimization::Reason_range_check, CHECK);
	  }

  {- -------------------------------------------
  (1) 例外オブジェクトにセットするメッセージを作成.
      ---------------------------------------- -}

	  // create exception
	  sprintf(message, "%d", index);

  {- -------------------------------------------
  (1) THROW_MSG() で, 例外オブジェクトの生成と送出を行う.
      ---------------------------------------- -}

	  THROW_MSG(s, message);
	IRT_END
	
```


