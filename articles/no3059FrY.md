---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp

### 名前(function name)
```
void TemplateTable::invokevirtual_helper(Register index,
                                         Register recv,
                                         Register flags) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (なお, 以下では一時的に rax と rdx を使用する)
      ---------------------------------------- -}

	  // Uses temporary registers rax, rdx

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(index, recv, rax, rdx);
	
  {- -------------------------------------------
  (1) コード生成:
      「呼び出し対象のメソッドに final 修飾子が付いているかどうかを確認.
        付いていなければ notFinal ラベルに分岐.」
      ---------------------------------------- -}

	  // Test for an invoke of a final method
	  Label notFinal;
	  __ movl(rax, flags);
	  __ andl(rax, (1 << ConstantPoolCacheEntry::vfinalMethod));
	  __ jcc(Assembler::zero, notFinal);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Register method = index;  // method must be rbx
	  assert(method == rbx,
	         "methodOop must be rbx for interpreter calling convention");
	
  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていた場合のコード生成)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) コード生成: (verify)
        ---------------------------------------- -}

	  // do the call - the index is actually the method to call
	  __ verify_oop(method);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「receiver が NULL でないことを確認する.
          NULL の場合は NullPointerException (See: [here](no30592Qc.html) for details)」
        ---------------------------------------- -}

	  // It's final, need a null check here!
	  __ null_check(recv);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「method data pointer (mdp) の値を更新しておく」
        ---------------------------------------- -}

	  // profile this call
	  __ profile_final_call(rax);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  __ jump_from_interpreted(method, rax);
	
  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていた場合)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は, final 修飾子が付いていなかった場合のコード生成)
      ---------------------------------------- -}

	  __ bind(notFinal);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「receiver の klass フィールドが NULL でないことを確認した後, 
          クラスオブジェクトを Rrecv にロードする.
          NULL の場合は NullPointerException (See: [here](no30592Qc.html) for details)」
        ---------------------------------------- -}

	  // get receiver klass
	  __ null_check(recv, oopDesc::klass_offset_in_bytes());
	  __ load_klass(rax, recv);
	
    {- -------------------------------------------
  (1.1) コード生成: (verify)
        ---------------------------------------- -}

	  __ verify_oop(rax);
	
    {- -------------------------------------------
  (1.1) コード生成:
        「method data pointer (mdp) の値を更新しておく」
        ---------------------------------------- -}

	  // profile this call
	  __ profile_virtual_call(rax, r14, rdx);
	
    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	  // get target methodOop & entry point
	  const int base = instanceKlass::vtable_start_offset() * wordSize;

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	  assert(vtableEntry::size() * wordSize == 8,
	         "adjust the scaling in the code below");

    {- -------------------------------------------
  (1.1) コード生成:
        「クラスオブジェクトから vtable を取得し, 
          CPCache から取得しておいた vtable 内の index 番号とつき合わせて methodOop を取得する.
          取得した結果は, 引数の method で指定されたレジスタに格納.」
        ---------------------------------------- -}

	  __ movptr(method, Address(rax, index,
	                                 Address::times_8,
	                                 base + vtableEntry::method_offset_in_bytes()));

    {- -------------------------------------------
  (1.1) コード生成:
        「?? (ここのロードは何か意味があるか?? 使われていないような... しかも参照先は interpreter_entry だし...)」
        ---------------------------------------- -}

	  __ movptr(rdx, Address(method, methodOopDesc::interpreter_entry_offset()));

    {- -------------------------------------------
  (1.1) コード生成:
        「実際の呼び出し処理を行う」
        ---------------------------------------- -}

	  __ jump_from_interpreted(method, rdx);
	}

  {- -------------------------------------------
  (1) (ここまでが, final 修飾子が付いていなかった場合)
      ---------------------------------------- -}
		
```


