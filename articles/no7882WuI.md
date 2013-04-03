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
void TemplateTable::fast_storefield(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(state, vtos);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ByteSize base = constantPoolCacheOopDesc::base_offset();
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  jvmti_post_fast_field_mod();
	
  {- -------------------------------------------
  (1) InterpreterMacroAssembler::get_cache_and_index_at_bcp() を呼んで, 以下のようなコードを生成.
      「現在箇所のバイトコード(bcp)に対応する CPCache のアドレスを rcx に, 対応する index を rbx に取得.」
      ---------------------------------------- -}

	  // access constant pool cache
	  __ get_cache_and_index_at_bcp(rcx, rbx, 1);
	
  {- -------------------------------------------
  (1) コード生成:
      「取得した CPCache のアドレスと index をもとに, フラグ情報を rdx に取得する. また, 実際のオフセット情報を rbx に取得する.」
      ---------------------------------------- -}

	  // test for volatile with rdx
	  __ movl(rdx, Address(rcx, rbx, Address::times_8,
	                       in_bytes(base +
	                                ConstantPoolCacheEntry::flags_offset())));
	
	  // replace index with field offset from cache entry
	  __ movptr(rbx, Address(rcx, rbx, Address::times_8,
	                         in_bytes(base + ConstantPoolCacheEntry::f2_offset())));
	
  {- -------------------------------------------
  (1) (なお, 現状の x86 ではメモリバリアは不要なので張っていない)
      ---------------------------------------- -}

	  // [jk] not needed currently
	  // volatile_barrier(Assembler::Membar_mask_bits(Assembler::LoadStore |
	  //                                              Assembler::StoreStore));
	
  {- -------------------------------------------
  (1) ここに「取得したフラグ情報の中から volatile 情報を取得して rdx に格納する」というコードを生成しておく (この情報は後で使用する).
      ---------------------------------------- -}

	  Label notVolatile;
	  __ shrl(rdx, ConstantPoolCacheEntry::volatileField);
	  __ andl(rdx, 0x1);
	
  {- -------------------------------------------
  (1) コード生成: (NULL チェック) & (verify)
      ---------------------------------------- -}

	  // Get object from stack
	  pop_and_check_object(rcx);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // field address
	  const Address field(rcx, rbx, Address::times_1);
	
  {- -------------------------------------------
  (1) コード生成:
      (バイトコードの型に応じた適切なストア命令を生成)
      ---------------------------------------- -}

	  // access field
	  switch (bytecode()) {
	  case Bytecodes::_fast_aputfield:
	    do_oop_store(_masm, field, rax, _bs->kind(), false);
	    break;
	  case Bytecodes::_fast_lputfield:
	    __ movq(field, rax);
	    break;
	  case Bytecodes::_fast_iputfield:
	    __ movl(field, rax);
	    break;
	  case Bytecodes::_fast_bputfield:
	    __ movb(field, rax);
	    break;
	  case Bytecodes::_fast_sputfield:
	    // fall through
	  case Bytecodes::_fast_cputfield:
	    __ movw(field, rax);
	    break;
	  case Bytecodes::_fast_fputfield:
	    __ movflt(field, xmm0);
	    break;
	  case Bytecodes::_fast_dputfield:
	    __ movdbl(field, xmm0);
	    break;
	  default:
	    ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) ここで, volatile field だった場合のメモリバリア処理を行う.
  
      以下のようなコードを生成.
      「volatile かどうかのチェック(testl(rdx, rdx))を行い, 
       もし volatile であれば, volatile_barrier() が生成したメモリバリアコードを実行する.
       volatile でなければ何もしない (notVolatile までジャンプする)」
      ---------------------------------------- -}

	  // Check for volatile store
	  __ testl(rdx, rdx);
	  __ jcc(Assembler::zero, notVolatile);
	  volatile_barrier(Assembler::Membar_mask_bits(Assembler::StoreLoad |
	                                               Assembler::StoreStore));
	  __ bind(notVolatile);
	}
	
```


