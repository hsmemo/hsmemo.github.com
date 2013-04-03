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
void TemplateTable::putfield_or_static(int byte_no, bool is_static) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(vtos, vtos);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Register cache = rcx;
	  const Register index = rdx;
	  const Register obj   = rcx;
	  const Register off   = rbx;
	  const Register flags = rax;
	  const Register bc    = c_rarg3;
	
  {- -------------------------------------------
  (1) TemplateTable::resolve_cache_and_index() を呼んで, 以下のようなコードを生成.
      「現在箇所のバイトコード(bcp)に対応する CPCache エントリのアドレスを rcx(以下の cache) に取得する.
      もし対応する CPCache エントリがまだ生成されていなければ, その生成も行う.」
      ---------------------------------------- -}

	  resolve_cache_and_index(byte_no, noreg, cache, index, sizeof(u2));

  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  jvmti_post_field_mod(cache, index, is_static);

  {- -------------------------------------------
  (1) TemplateTable::load_field_cp_cache_entry() を呼んで, 以下のようなコードを生成.
      「取得した CPCache エントリの中から, 実際のオフセット情報を rbx(以下の off) に取得.
      また, 取得した CPCache エントリの中から, フラグ情報を rax(以下の flags) に取得.」
      ---------------------------------------- -}

	  load_field_cp_cache_entry(obj, cache, index, off, flags, is_static);
	
  {- -------------------------------------------
  (1) (現状の x86 では不要なので, メモリバリアは張っていない)
      ---------------------------------------- -}

	  // [jk] not needed currently
	  // volatile_barrier(Assembler::Membar_mask_bits(Assembler::LoadStore |
	  //                                              Assembler::StoreStore));
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label notVolatile, Done;

  {- -------------------------------------------
  (1) ここに「取得したフラグ情報の中から volatile 情報を取得して rdx に格納する」というコードを生成しておく (この情報は後で使用する).
      ---------------------------------------- -}

	  __ movl(rdx, flags);
	  __ shrl(rdx, ConstantPoolCacheEntry::volatileField);
	  __ andl(rdx, 0x1);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // field address
	  const Address field(obj, off, Address::times_1);
	
	  Label notByte, notInt, notShort, notChar,
	        notLong, notFloat, notObj, notDouble;
	
  {- -------------------------------------------
  (1) CPCache のエントリから型情報を取り出す.
      ---------------------------------------- -}

	  __ shrl(flags, ConstantPoolCacheEntry::tosBits);
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(btos == 0, "change code, btos != 0");

  {- -------------------------------------------
  (1) 以下, 型情報に応じた処理を行う.
      (1) まず btos かどうかをチェック.
      (1) 次に atos かどうかをチェック.
      (1) 次に itos かどうかをチェック.
      (1) 次に ctos かどうかをチェック.
      (1) 次に stos かどうかをチェック.
      (1) 次に ltos かどうかをチェック.
      (1) 次に ftos かどうかをチェック.
      (1) 以上のどれでもなければ dtos.
          
      なお, それぞれの場合で高速版への rewrite 処理を行っている (See: [here](no7882vBO.html) for details)
      そのため, ここのコードはそれぞれの箇所で最初の一回だけ実行される.
      (なぜ static は最適化しない?? static は使われる比率が小さいから効果が低い?? #TODO)
      ---------------------------------------- -}

    {- -------------------------------------------
  (1.1) (ここが btos チェック)
        ---------------------------------------- -}

	  __ andl(flags, 0x0f);
	  __ jcc(Assembler::notZero, notByte);

    {- -------------------------------------------
  (1.1) (ここが btos の場合の処理)
        ---------------------------------------- -}

	  // btos
	  __ pop(btos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movb(field, rax);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_bputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが atos チェック)
        ---------------------------------------- -}

	  __ bind(notByte);
	  __ cmpl(flags, atos);
	  __ jcc(Assembler::notEqual, notObj);

    {- -------------------------------------------
  (1.1) (ここが atos の場合の処理)
        ---------------------------------------- -}

	  // atos
	  __ pop(atos);
	  if (!is_static) pop_and_check_object(obj);
	
	  // Store into the field
	  do_oop_store(_masm, field, rax, _bs->kind(), false);
	
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_aputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが itos チェック)
        ---------------------------------------- -}

	  __ bind(notObj);
	  __ cmpl(flags, itos);
	  __ jcc(Assembler::notEqual, notInt);

    {- -------------------------------------------
  (1.1) (ここが itos の場合の処理)
        ---------------------------------------- -}

	  // itos
	  __ pop(itos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movl(field, rax);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_iputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが ctos チェック)
        ---------------------------------------- -}

	  __ bind(notInt);
	  __ cmpl(flags, ctos);
	  __ jcc(Assembler::notEqual, notChar);

    {- -------------------------------------------
  (1.1) (ここが ctos の場合の処理)
        ---------------------------------------- -}

	  // ctos
	  __ pop(ctos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movw(field, rax);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_cputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが stos チェック)
        ---------------------------------------- -}

	  __ bind(notChar);
	  __ cmpl(flags, stos);
	  __ jcc(Assembler::notEqual, notShort);

    {- -------------------------------------------
  (1.1) (ここが stos の場合の処理)
        ---------------------------------------- -}

	  // stos
	  __ pop(stos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movw(field, rax);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_sputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが ltos チェック)
        ---------------------------------------- -}

	  __ bind(notShort);
	  __ cmpl(flags, ltos);
	  __ jcc(Assembler::notEqual, notLong);

    {- -------------------------------------------
  (1.1) (ここが ltos の場合の処理)
        ---------------------------------------- -}

	  // ltos
	  __ pop(ltos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movq(field, rax);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_lputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (ここが ftos チェック)
        ---------------------------------------- -}

	  __ bind(notLong);
	  __ cmpl(flags, ftos);
	  __ jcc(Assembler::notEqual, notFloat);

    {- -------------------------------------------
  (1.1) (ここが ftos の場合の処理)
        ---------------------------------------- -}

	  // ftos
	  __ pop(ftos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movflt(field, xmm0);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_fputfield, bc, rbx);
	  }
	  __ jmp(Done);
	
    {- -------------------------------------------
  (1.1) (以上のどれでもなければ dtos)
        ---------------------------------------- -}

	  __ bind(notFloat);
	#ifdef ASSERT
	  __ cmpl(flags, dtos);
	  __ jcc(Assembler::notEqual, notDouble);
	#endif

    {- -------------------------------------------
  (1.1) (ここが dtos の場合の処理)
        ---------------------------------------- -}

	  // dtos
	  __ pop(dtos);
	  if (!is_static) pop_and_check_object(obj);
	  __ movdbl(field, xmm0);
	  if (!is_static) {
	    patch_bytecode(Bytecodes::_fast_dputfield, bc, rbx);
	  }
	
	#ifdef ASSERT
	  __ jmp(Done);
	
	  __ bind(notDouble);
	  __ stop("Bad state");
	#endif
	
  {- -------------------------------------------
  (1) ここで, volatile field だった場合のメモリバリア処理を行う.
  
      以下のようなコードを生成.
      「volatile かどうかのチェック(testl(rdx, rdx))を行い, 
       もし volatile であれば, volatile_barrier() が生成したメモリバリアコードを実行する.
       volatile でなければ何もしない (notVolatile までジャンプする)」
      ---------------------------------------- -}

	  __ bind(Done);
	  // Check for volatile store
	  __ testl(rdx, rdx);
	  __ jcc(Assembler::zero, notVolatile);
	  volatile_barrier(Assembler::Membar_mask_bits(Assembler::StoreLoad |
	                                               Assembler::StoreStore));
	
	  __ bind(notVolatile);
	}
	
```


