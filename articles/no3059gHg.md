---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_return_entry_for(TosState state, int step) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  TosState incoming_state = state;
	
	  Label cont;

  {- -------------------------------------------
  (1) ?? (どこからも使われていないような... #TODO)
      ---------------------------------------- -}

	  address compiled_entry = __ pc();
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) コード生成: (ただし, 32bit 環境でかつ C2 利用時でない場合には, 不要なので生成しない)
      「32bit 環境では long 値だけは扱いが特殊なので調整しておく.」
      (正確に言うと, 32bit 環境では C2 とインタープリタで使うレジスタがずれている.
       C2 は G1 を使うがインタープリタは O0 と O1 を使う.)
      ---------------------------------------- -}

	#if !defined(_LP64) && defined(COMPILER2)
	  // All return values are where we want them, except for Longs.  C2 returns
	  // longs in G1 in the 32-bit build whereas the interpreter wants them in O0/O1.
	  // Since the interpreter will return longs in G1 and O0/O1 in the 32bit
	  // build even if we are returning from interpreted we just do a little
	  // stupid shuffing.
	  // Note: I tried to make c2 return longs in O0/O1 and G1 so we wouldn't have to
	  // do this here. Unfortunately if we did a rethrow we'd see an machepilog node
	  // first which would move g1 -> O0/O1 and destroy the exception we were throwing.
	
	  if (incoming_state == ltos) {
	    __ srl (G1,  0, O1);
	    __ srlx(G1, 32, O0);
	  }
	#endif // !_LP64 && COMPILER2
	
  {- -------------------------------------------
  (1) ?? (この cont ラベルはどこからも使われていないような... #TODO)
      ---------------------------------------- -}

	  __ bind(cont);
	
  {- -------------------------------------------
  (1) コード生成:
      「SP を Llast_SP の値に復帰させる」
      ---------------------------------------- -}

	  // The callee returns with the stack possibly adjusted by adapter transition
	  // We remove that possible adjustment here.
	  // All interpreter local registers are untouched. Any result is passed back
	  // in the O0/O1 or float registers. Before continuing, the arguments must be
	  // popped from the java expression stack; i.e., Lesp must be adjusted.
	
	  __ mov(Llast_SP, SP);   // Remove any adapter added stack space.
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label L_got_cache, L_giant_index;
	  const Register cache = G3_scratch;
	  const Register size  = G1_scratch;

  {- -------------------------------------------
  (1) コード生成: (ただし, EnableInvokeDynamic オプションが指定されていない場合には, 不要なので生成しない)
      「もしリターンしてきたのが invokedynamic で呼び出されたメソッドからだった場合, 
        L_giant_index にジャンプする.
  
       (L_giant_index は, get_cache_and_index_at_bcp() で CPCache entry を取得する際に, 
        通常よりも大きな index size を使う処理パス.
        CPCache entry を取得した後は元のパスに合流する.)」
      ---------------------------------------- -}

	  if (EnableInvokeDynamic) {
	    __ ldub(Address(Lbcp, 0), G1_scratch);  // Load current bytecode.
	    __ cmp(G1_scratch, Bytecodes::_invokedynamic);
	    __ br(Assembler::equal, false, Assembler::pn, L_giant_index);
	    __ delayed()->nop();
	  }

  {- -------------------------------------------
  (1) コード生成:
      「Lesp レジスタの値を元に戻す
        (メソッド呼び出し後には積んであった引数は消すべきなので, 
         引数の量を計算し, その分だけ Lesp の値を増やす)」
      ---------------------------------------- -}

	  __ get_cache_and_index_at_bcp(cache, G1_scratch, 1);
	  __ bind(L_got_cache);
	  __ ld_ptr(cache, constantPoolCacheOopDesc::base_offset() +
	                   ConstantPoolCacheEntry::flags_offset(), size);
	  __ and3(size, 0xFF, size);                   // argument size in words
	  __ sll(size, Interpreter::logStackElementSize, size); // each argument size in bytes
	  __ add(Lesp, size, Lesp);                    // pop arguments

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
	    __ get_cache_and_index_at_bcp(cache, G1_scratch, 1, sizeof(u4));
	    __ ba(false, L_got_cache);
	    __ delayed()->nop();
	  }
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


