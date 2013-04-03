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
void TemplateTable::prepare_invoke(Register method, Register index, int byte_no) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // determine flags
	  Bytecodes::Code code = bytecode();
	  const bool is_invokeinterface  = code == Bytecodes::_invokeinterface;
	  const bool is_invokedynamic    = code == Bytecodes::_invokedynamic;
	  const bool is_invokevirtual    = code == Bytecodes::_invokevirtual;
	  const bool is_invokespecial    = code == Bytecodes::_invokespecial;
	  const bool load_receiver      = (code != Bytecodes::_invokestatic && code != Bytecodes::_invokedynamic);
	  const bool receiver_null_check = is_invokespecial;
	  const bool save_flags = is_invokeinterface || is_invokevirtual;
	  // setup registers & access constant pool cache
	  const Register recv   = rcx;
	  const Register flags  = rdx;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(method, index, recv, flags);
	
  {- -------------------------------------------
  (1) コード生成:
      「r13(= bcp, リターン後に実行すべき bytecode のアドレス)を退避しておく」
      ---------------------------------------- -}

	  // save 'interpreter return address'
	  __ save_bcp();
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する methodOop の情報を CPCache から取得し, 引数の method で指定されたレジスタに格納する.
        (なお, この時点で resolve が終わってなければ resolve も行う)」
      ---------------------------------------- -}

	  load_invoke_cp_cache_entry(byte_no, method, index, flags, is_invokevirtual, false, is_invokedynamic);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, invokestatic と invokedynamic の場合には, 不要なので生成しない)
      「receiver のアドレスを O0 レジスタにロードする」
      ---------------------------------------- -}

	  // load receiver if needed (note: no return address pushed yet)
	  if (load_receiver) {
	    assert(!is_invokedynamic, "");
	    __ movl(recv, flags);
	    __ andl(recv, 0xFF);
	    Address recv_addr(rsp, recv, Address::times_8, -Interpreter::expr_offset_in_bytes(1));
	    __ movptr(recv, recv_addr);
	    __ verify_oop(recv);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, invokespecial 以外の場合には, 不要なので生成しない)
      「receiver が NULL でないことを確認する.
        NULL の場合は NullPointerException (See: [here](no30592Qc.html) for details)」
      ---------------------------------------- -}

	  // do null check if needed
	  if (receiver_null_check) {
	    __ null_check(recv);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, invokeinterface と invokevirtual 以外の場合には, 不要なので生成しない)
      「flags の値を, 一時的に r13 に待避しておく」
      ---------------------------------------- -}

	  if (save_flags) {
	    __ movl(r13, flags);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する return entry のアドレスを取得し, スタック上に push しておく」
  
       (なお, 引数に応じて取得する return entry がなる.
        * invokeinterface や invokedynamic を生成する場合
          5byte先のバイトコードにdispatchする return entry を取得
        * それ以外の場合
          3byte先のバイトコードにdispatchする return entry を取得)
      ---------------------------------------- -}

	  // compute return type
	  __ shrl(flags, ConstantPoolCacheEntry::tosBits);
	  // Make sure we don't need to mask flags for tosBits after the above shift
	  ConstantPoolCacheEntry::verify_tosBits();
	  // load return address
	  {
	    address table_addr;
	    if (is_invokeinterface || is_invokedynamic)
	      table_addr = (address)Interpreter::return_5_addrs_by_index_table();
	    else
	      table_addr = (address)Interpreter::return_3_addrs_by_index_table();
	    ExternalAddress table(table_addr);
	    __ lea(rscratch1, table);
	    __ movptr(flags, Address(rscratch1, flags, Address::times_ptr));
	  }
	
	  // push return address
	  __ push(flags);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, invokeinterface と invokevirtual 以外の場合には, 不要なので生成しない)
      「flags の値を, r13 から復帰させる.
        また r13 (bcp) の値も復帰させておく.」
      ---------------------------------------- -}

	  // Restore flag field from the constant pool cache, and restore esi
	  // for later null checks.  r13 is the bytecode pointer
	  if (save_flags) {
	    __ movl(flags, r13);
	    __ restore_bcp();
	  }
	}
	
```


