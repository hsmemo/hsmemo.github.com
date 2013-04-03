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
void TemplateTable::resolve_cache_and_index(int byte_no,
                                            Register result,
                                            Register Rcache,
                                            Register index,
                                            size_t index_size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const Register temp = rbx;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(result, Rcache, index, temp);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label resolved;

  {- -------------------------------------------
  (1) InterpreterMacroAssembler::get_cache_and_index_at_bcp() を呼んで, 以下のようなコードを生成.
      「現在箇所のバイトコード(bcp)に対応する CPCache エントリのアドレスを Rcache に取得.
      (ただし, まだ対応する CPCache エントリのアドレスを作っていなかった場合は, ...#TODO が取得される)」
      ---------------------------------------- -}

	  __ get_cache_and_index_at_bcp(Rcache, index, 1, index_size);

  {- -------------------------------------------
  (1) 引数の byte_no に応じて以下の 2通りのコードを生成する.
      どちらの場合も, ...#TODO
      * f1_oop の場合: 
        「取得した CPCache エントリの f1 フィールドをロードし, NULL かどうかチェックする.
        NULL でなければ(= 既にそのエントリは作成済みであれば), resolved ラベルまでジャンプする.
        そうでなければ, このままフォールスルーして作成処理を行う.」
      * それ以外の場合 (f1_byte または f2_byte): 
        「取得した CPCache エントリの ...  かどうかチェックする. #TODO
        同じであれば(= 既にそのエントリは作成済みであれば), resolved ラベルまでジャンプする.
        そうでなければ, このままフォールスルーして作成処理を行う.」
      ---------------------------------------- -}

	  if (byte_no == f1_oop) {
	    // We are resolved if the f1 field contains a non-null object (CallSite, etc.)
	    // This kind of CP cache entry does not need to match the flags byte, because
	    // there is a 1-1 relation between bytecode type and CP entry type.
	    assert(result != noreg, ""); //else do cmpptr(Address(...), (int32_t) NULL_WORD)
	    __ movptr(result, Address(Rcache, index, Address::times_ptr, constantPoolCacheOopDesc::base_offset() + ConstantPoolCacheEntry::f1_offset()));
	    __ testptr(result, result);
	    __ jcc(Assembler::notEqual, resolved);
	  } else {
	    assert(byte_no == f1_byte || byte_no == f2_byte, "byte_no out of range");
	    assert(result == noreg, "");  //else change code for setting result
	    const int shift_count = (1 + byte_no) * BitsPerByte;
	    __ movl(temp, Address(Rcache, index, Address::times_ptr, constantPoolCacheOopDesc::base_offset() + ConstantPoolCacheEntry::indices_offset()));
	    __ shrl(temp, shift_count);
	    // have we resolved this bytecode?
	    __ andl(temp, 0xFF);
	    __ cmpl(temp, (int) bytecode());
	    __ jcc(Assembler::equal, resolved);
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (entry は, まだ CPCache エントリが生成されてなかった場合に呼び出すべき関数 (CPCache エントリ生成用の関数).
       以下の通り, 現在箇所のバイトコードに応じて呼び出す関数は変わる.)
      ---------------------------------------- -}

	  // resolve first time through
	  address entry;
	  switch (bytecode()) {
	  case Bytecodes::_getstatic:
	  case Bytecodes::_putstatic:
	  case Bytecodes::_getfield:
	  case Bytecodes::_putfield:
	    entry = CAST_FROM_FN_PTR(address, InterpreterRuntime::resolve_get_put);
	    break;
	  case Bytecodes::_invokevirtual:
	  case Bytecodes::_invokespecial:
	  case Bytecodes::_invokestatic:
	  case Bytecodes::_invokeinterface:
	    entry = CAST_FROM_FN_PTR(address, InterpreterRuntime::resolve_invoke);
	    break;
	  case Bytecodes::_invokedynamic:
	    entry = CAST_FROM_FN_PTR(address, InterpreterRuntime::resolve_invokedynamic);
	    break;
	  case Bytecodes::_fast_aldc:
	    entry = CAST_FROM_FN_PTR(address, InterpreterRuntime::resolve_ldc);
	    break;
	  case Bytecodes::_fast_aldc_w:
	    entry = CAST_FROM_FN_PTR(address, InterpreterRuntime::resolve_ldc);
	    break;
	  default:
	    ShouldNotReachHere();
	    break;
	  }

  {- -------------------------------------------
  (1) コード生成:
      「CPCache エントリ生成用の関数(以下の entry)を呼び出す.
       その後, 改めて get_cache_and_index_at_bcp() が生成したコードを実行し, 
      現在箇所のバイトコード(bcp)に対応する CPCache エントリのアドレスを Rcache に取得する.」
      
      (もし...#TODO であれば, 以下のコードも生成しておく.
      「取得しなおした CPCache エントリの f1 フィールドを, result レジスタにロードする.」)
      ---------------------------------------- -}

	  __ movl(temp, (int) bytecode());
	  __ call_VM(noreg, entry, temp);
	
	  // Update registers with resolved info
	  __ get_cache_and_index_at_bcp(Rcache, index, 1, index_size);
	  if (result != noreg)
	    __ movptr(result, Address(Rcache, index, Address::times_ptr, constantPoolCacheOopDesc::base_offset() + ConstantPoolCacheEntry::f1_offset()));

  {- -------------------------------------------
  (1) (ここが resolved ラベルの位置.
       CPCache エントリが既に生成されていた場合は, この地点まで飛んでくる)
      ---------------------------------------- -}

	  __ bind(resolved);
	}
	
```


