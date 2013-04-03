---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)

```
// Empty method, generate a very fast return.

```

### 名前(function name)
```
address InterpreterGenerator::generate_empty_entry(void) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (空のメソッドなので, return する以外にすることはない...)
      ---------------------------------------- -}

	  // A method that does nother but return...
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label slow_path;
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(G5_method);
	
  {- -------------------------------------------
  (1) UseFastEmptyMethods オプションが指定されている場合には, 
      以下のコードを生成し, そのエントリポイントのアドレスをリターンする.
      (UseFastEmptyMethods オプションが指定されていない場合には, 単に NULL をリターンするだけ)
  
      (<= ところで, 何故少し作ってしまってから判定してるんだろう?? コードを作る前に判定した方がいいのでは... #TODO)
      ---------------------------------------- -}

	  // do nothing for empty methods (do not even increment invocation counter)
	  if ( UseFastEmptyMethods) {

    {- -------------------------------------------
  (1.1) コード生成:
        「Safepoint 処理中かどうかをチェックする (= SafepointSynchronize::address_of_state を確認).
          もし Safepoint 停止の要求が出ていれば, slow_path ラベルにジャンプして停止処理を行う.」
        ---------------------------------------- -}

	    // If we need a safepoint check, generate full interpreter entry.
	    AddressLiteral sync_state(SafepointSynchronize::address_of_state());
	    __ set(sync_state, G3_scratch);
	    __ cmp(G3_scratch, SafepointSynchronize::_not_synchronized);
	    __ br(Assembler::notEqual, false, Assembler::pn, slow_path);
	    __ delayed()->nop();
	
    {- -------------------------------------------
  (1.1) コード生成:
        「Safepoint 停止要求が出ていなければ, 単にリターンするだけ.
          (あわせて, SP を O5_savedSP の値に復帰させておく)」
        ---------------------------------------- -}

	    // Code: _return
	    __ retl();
	    __ delayed()->mov(O5_savedSP, SP);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「Safepoint 停止要求が出ていた場合には, 
         InterpreterGenerator::generate_normal_entry() が生成する通常パスにフォールバック」
        ---------------------------------------- -}

	    __ bind(slow_path);
	    (void) generate_normal_entry(false);
	
	    return entry;
	  }
	  return NULL;
	}
	
```


