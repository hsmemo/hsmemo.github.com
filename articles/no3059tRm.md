---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_return_entry_for(TosState state, int step) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();
	
  {- -------------------------------------------
  (1) コード生成:
      「SP をフレーム上に待避しておいた値に復帰させる.
        (復帰後に, フレーム上の待避箇所は 0 でクリアしておく)」
      ---------------------------------------- -}

	  // Restore stack bottom in case i2c adjusted stack
	  __ movptr(rsp, Address(rbp, frame::interpreter_frame_last_sp_offset * wordSize));
	  // and NULL it as marker that esp is now tos until next java call
	  __ movptr(Address(rbp, frame::interpreter_frame_last_sp_offset * wordSize), (int32_t)NULL_WORD);
	
  {- -------------------------------------------
  (1) コード生成:
      「r13(bcp)や r14(locals) を元の値に復帰させる」
      ---------------------------------------- -}

	  __ restore_bcp();
	  __ restore_locals();
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label L_got_cache, L_giant_index;

  {- -------------------------------------------
  (1) コード生成: (ただし, EnableInvokeDynamic オプションが指定されていない場合には, 不要なので生成しない)
      「もしリターンしてきたのが invokedynamic で呼び出されたメソッドからだった場合, 
        L_giant_index にジャンプする.
  
       (L_giant_index は, get_cache_and_index_at_bcp() で CPCache entry を取得する際に, 
        通常よりも大きな index size を使う処理パス.
        CPCache entry を取得した後は元のパスに合流する.)」
      ---------------------------------------- -}

	  if (EnableInvokeDynamic) {
	    __ cmpb(Address(r13, 0), Bytecodes::_invokedynamic);
	    __ jcc(Assembler::equal, L_giant_index);
	  }

  {- -------------------------------------------
  (1) コード生成:
      「SP の値を元に戻す
        (メソッド呼び出し後には積んであった引数は消すべきなので, 
         引数の量を計算し, その分だけ rsp の値を増やす)」
      ---------------------------------------- -}

	  __ get_cache_and_index_at_bcp(rbx, rcx, 1, sizeof(u2));
	  __ bind(L_got_cache);
	  __ movl(rbx, Address(rbx, rcx,
	                       Address::times_ptr,
	                       in_bytes(constantPoolCacheOopDesc::base_offset()) +
	                       3 * wordSize));
	  __ andl(rbx, 0xFF);
	  __ lea(rsp, Address(rsp, rbx, Address::times_8));

  {- -------------------------------------------
  (1) コード生成:
      「dispatch_next() が生成するコードで, 
        メソッド呼び出しの次のバイトコードに対応するテンプレートへとジャンプする」
      ---------------------------------------- -}

	  __ dispatch_next(state, step);
	
  {- -------------------------------------------
  (1) コード生成: (ただし, EnableInvokeDynamic オプションが指定されていない場合には, 不要なので生成しない)
      「L_giant_index ラベルに対応する飛び先を生成する.
  
        get_cache_and_index_at_bcp() が生成するコードで 
        CPCache entry を取得した後, 
        上の L_got_cache 以降のパスに合流」
      ---------------------------------------- -}

	  // out of the main line of code...
	  if (EnableInvokeDynamic) {
	    __ bind(L_giant_index);
	    __ get_cache_and_index_at_bcp(rbx, rcx, 1, sizeof(u4));
	    __ jmp(L_got_cache);
	  }
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


