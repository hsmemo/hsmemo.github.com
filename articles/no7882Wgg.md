---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::fast_accessfield(TosState state) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(atos, state);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rcache  = G3_scratch;
	  Register index   = G4_scratch;
	  Register Roffset = G4_scratch;
	  Register Rflags  = Rcache;
	  ByteSize cp_base_offset = constantPoolCacheOopDesc::base_offset();
	
  {- -------------------------------------------
  (1) InterpreterMacroAssembler::get_cache_and_index_at_bcp() を呼んで, 以下のようなコードを生成.
      「現在箇所のバイトコード(bcp)に対応する CPCache エントリのアドレスを Rcache に取得」
      ---------------------------------------- -}

	  __ get_cache_and_index_at_bcp(Rcache, index, 1);

  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  jvmti_post_field_access(Rcache, index, /*is_static*/false, /*has_tos*/true);
	
  {- -------------------------------------------
  (1) コード生成:
      「取得した CPCache エントリの中から, 実際のオフセット情報を Roffset に取得」
      ---------------------------------------- -}

	  __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::f2_offset(), Roffset);
	
  {- -------------------------------------------
  (1) コード生成: (NULL チェック)
      ---------------------------------------- -}

	  __ null_check(Otos_i);

  {- -------------------------------------------
  (1) コード生成: (verify)
      ---------------------------------------- -}

	  __ verify_oop(Otos_i);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label exit;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (メモリバリアの必要がある環境かどうかを示す)
      ---------------------------------------- -}

	  Assembler::Membar_mask_bits membar_bits =
	    Assembler::Membar_mask_bits(Assembler::LoadLoad | Assembler::LoadStore);

  {- -------------------------------------------
  (1) もしメモリバリアの必要がある環境であれば, 
      ここに「volatile 情報を Lscratch に取得する」というコードを生成しておく (この情報は後で使用する).
      (ちなみに, このコードによると, 
      volatile 情報は, CPCache エントリの ConstantPoolCacheEntry::volatileField ビットの箇所に入っている模様)
      ---------------------------------------- -}

	  if (__ membar_has_effect(membar_bits)) {
	    // Get volatile flag
	    __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::f2_offset(), Rflags);
	    __ set((1 << ConstantPoolCacheEntry::volatileField), Lscratch);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      (バイトコードの型に応じた適切なロード命令を生成)
      ---------------------------------------- -}

	  switch (bytecode()) {
	    case Bytecodes::_fast_bgetfield:
	      __ ldsb(Otos_i, Roffset, Otos_i);
	      break;
	    case Bytecodes::_fast_cgetfield:
	      __ lduh(Otos_i, Roffset, Otos_i);
	      break;
	    case Bytecodes::_fast_sgetfield:
	      __ ldsh(Otos_i, Roffset, Otos_i);
	      break;
	    case Bytecodes::_fast_igetfield:
	      __ ld(Otos_i, Roffset, Otos_i);
	      break;
	    case Bytecodes::_fast_lgetfield:
	      __ ld_long(Otos_i, Roffset, Otos_l);
	      break;
	    case Bytecodes::_fast_fgetfield:
	      __ ldf(FloatRegisterImpl::S, Otos_i, Roffset, Ftos_f);
	      break;
	    case Bytecodes::_fast_dgetfield:
	      __ ldf(FloatRegisterImpl::D, Otos_i, Roffset, Ftos_d);
	      break;
	    case Bytecodes::_fast_agetfield:
	      __ load_heap_oop(Otos_i, Roffset, Otos_i);
	      break;
	    default:
	      ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) ここで, volatile field だった場合のメモリバリア処理を行う (メモリバリアが必要な環境の場合).
      
      以下のようなコードを生成.
      「volatile かどうかのチェック(btst(Lscratch, Rflags))を行い, 
       もし volatile であれば, volatile_barrier() が生成したメモリバリアコードを実行する.
       volatile でなければ何もしない (exit までジャンプする)」
      ---------------------------------------- -}

	  if (__ membar_has_effect(membar_bits)) {
	    __ btst(Lscratch, Rflags);
	    __ br(Assembler::zero, false, Assembler::pt, exit);
	    __ delayed()->nop();
	    volatile_barrier(membar_bits);
	    __ bind(exit);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (verify)
      (atos の場合には verify_oop() しておく)
      ---------------------------------------- -}

	  if (state == atos) {
	    __ verify_oop(Otos_i);    // does not blow flags!
	  }
	}
	
```


