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

	  Register Rcache = G3_scratch;
	  Register Rclass = Rcache;
	  Register Roffset= G4_scratch;
	  Register Rflags = G1_scratch;
	  ByteSize cp_base_offset = constantPoolCacheOopDesc::base_offset();
	
  {- -------------------------------------------
  (1) コード生成: (JVMTI のフック点)
      ---------------------------------------- -}

	  jvmti_post_fast_field_mod();
	
  {- -------------------------------------------
  (1) InterpreterMacroAssembler::get_cache_and_index_at_bcp() を呼んで, 以下のようなコードを生成.
      「現在箇所のバイトコード(bcp)に対応する CPCache エントリのアドレスを Rcache に取得」
      ---------------------------------------- -}

	  __ get_cache_and_index_at_bcp(Rcache, G4_scratch, 1);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (メモリバリアの必要がある環境かどうかを示す)
      ---------------------------------------- -}

	  Assembler::Membar_mask_bits read_bits =
	    Assembler::Membar_mask_bits(Assembler::LoadStore | Assembler::StoreStore);
	  Assembler::Membar_mask_bits write_bits = Assembler::StoreLoad;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label notVolatile, checkVolatile, exit;

  {- -------------------------------------------
  (1) もしメモリバリアの必要がある環境であれば, 
      ここに「volatile 情報を Lscratch に取得する」というコードを生成しておく (この情報は後で使用する).
      (ちなみに, このコードによると, 
      volatile 情報は, CPCache エントリの ConstantPoolCacheEntry::volatileField ビットの箇所に入っている模様)
      
      さらに, メモリバリアが必要な環境の中でも read で必要な場合には, 
      ここに volatile field だった場合のメモリバリア処理も入れておく.
      以下のようなコードを生成.
      「volatile かどうかのチェック(tst(Lscratch))を行い, 
       もし volatile であれば, volatile_barrier() が生成したメモリバリアコードを実行する.
       volatile でなければ何もしない (notVolatile までジャンプする)」
      ---------------------------------------- -}

	  if (__ membar_has_effect(read_bits) || __ membar_has_effect(write_bits)) {
	    __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::flags_offset(), Rflags);
	    __ set((1 << ConstantPoolCacheEntry::volatileField), Lscratch);
	    __ and3(Rflags, Lscratch, Lscratch);
	    if (__ membar_has_effect(read_bits)) {
	      __ tst(Lscratch);
	      __ br(Assembler::zero, false, Assembler::pt, notVolatile);
	      __ delayed()->nop();
	      volatile_barrier(read_bits);
	      __ bind(notVolatile);
	    }
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「取得した CPCache エントリの中から, 実際のオフセット情報を Roffset に取得」
      ---------------------------------------- -}

	  __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::f2_offset(), Roffset);

  {- -------------------------------------------
  (1) コード生成: (NULL チェック) & (verify)
      ---------------------------------------- -}

	  pop_and_check_object(Rclass);
	
  {- -------------------------------------------
  (1) コード生成:
      (バイトコードの型に応じた適切なストア命令を生成)
      ---------------------------------------- -}

	  switch (bytecode()) {
	    case Bytecodes::_fast_bputfield: __ stb(Otos_i, Rclass, Roffset); break;
	    case Bytecodes::_fast_cputfield: /* fall through */
	    case Bytecodes::_fast_sputfield: __ sth(Otos_i, Rclass, Roffset); break;
	    case Bytecodes::_fast_iputfield: __ st(Otos_i, Rclass, Roffset);  break;
	    case Bytecodes::_fast_lputfield: __ st_long(Otos_l, Rclass, Roffset); break;
	    case Bytecodes::_fast_fputfield:
	      __ stf(FloatRegisterImpl::S, Ftos_f, Rclass, Roffset);
	      break;
	    case Bytecodes::_fast_dputfield:
	      __ stf(FloatRegisterImpl::D, Ftos_d, Rclass, Roffset);
	      break;
	    case Bytecodes::_fast_aputfield:
	      do_oop_store(_masm, Rclass, Roffset, 0, Otos_i, G1_scratch, _bs->kind(), false);
	      break;
	    default:
	      ShouldNotReachHere();
	  }
	
  {- -------------------------------------------
  (1) ここで, volatile field だった場合のメモリバリア処理を行う (メモリバリアが必要な環境の場合, かつ write 後のバリアが必要な場合).
  
      以下のようなコードを生成.
      「volatile かどうかのチェック(tst(Lscratch))を行い, 
       もし volatile であれば, volatile_barrier() が生成したメモリバリアコードを実行する.
       volatile でなければ何もしない (exit までジャンプする)」
      ---------------------------------------- -}

	  if (__ membar_has_effect(write_bits)) {
	    __ tst(Lscratch);
	    __ br(Assembler::zero, false, Assembler::pt, exit);
	    __ delayed()->nop();
	    volatile_barrier(Assembler::StoreLoad);
	    __ bind(exit);
	  }
	}
	
```


