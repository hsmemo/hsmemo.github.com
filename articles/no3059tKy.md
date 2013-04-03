---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreter_x86_64.cpp
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
  (1) (生成したコードが実行される時点では, レジスタには以下の値が入っているはず)
      ---------------------------------------- -}

	  // rbx: methodOop
	  // r13: sender sp must set sp to this value on return
	
  {- -------------------------------------------
  (1) UseFastEmptyMethods オプションが指定されていない場合には, 特にすることはない.
      単に NULL をリターンするだけ.
      ---------------------------------------- -}

	  if (!UseFastEmptyMethods) {
	    return NULL;
	  }
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry_point = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「Safepoint 処理中かどうかをチェックする (= SafepointSynchronize::address_of_state を確認).
        もし Safepoint 停止の要求が出ていれば, slow_path ラベルにジャンプして停止処理を行う.」
      ---------------------------------------- -}

	  // If we need a safepoint check, generate full interpreter entry.
	  Label slow_path;
	  __ cmp32(ExternalAddress(SafepointSynchronize::address_of_state()),
	           SafepointSynchronize::_not_synchronized);
	  __ jcc(Assembler::notEqual, slow_path);
	
  {- -------------------------------------------
  (1) コード生成:
      「Safepoint 停止要求が出ていなければ, 単にリターンするだけ.
        (あわせて, SP を r13 の値に復帰させておく)」
      ---------------------------------------- -}

	  // do nothing for empty methods (do not even increment invocation counter)
	  // Code: _return
	  // _return
	  // return w/o popping parameters
	  __ pop(rax);
	  __ mov(rsp, r13);
	  __ jmp(rax);
	
  {- -------------------------------------------
  (1) コード生成:
      「Safepoint 停止要求が出ていた場合には, 
       InterpreterGenerator::generate_normal_entry() が生成する通常パスにフォールバック」
      ---------------------------------------- -}

	  __ bind(slow_path);
	  (void) generate_normal_entry(false);

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry_point;
	
	}
	
```


