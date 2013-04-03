---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.cpp
### 説明(description)
// Look up the method for a megamorphic invokeinterface call.
// The target method is determined by <intf_klass, itable_index>.
// The receiver klass is in recv_klass.
// On success, the result will be in method_result, and execution falls through.
// On failure, execution transfers to the given label.


### 名前(function name)
```
void MacroAssembler::lookup_interface_method(Register recv_klass,
                                             Register intf_klass,
                                             RegisterOrConstant itable_index,
                                             Register method_result,
                                             Register scan_temp,
                                             Label& L_no_such_interface) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(recv_klass, intf_klass, method_result, scan_temp);
	  assert(itable_index.is_constant() || itable_index.as_register() == method_result,
	         "caller must use same register for non-constant itable index as for method");
	
  {- -------------------------------------------
  (1) (以下の処理で, まず対応する itableOffsetEntry を探す.
       なお, ここで行っている内容は, instanceKlass::method_at_itable() の処理とほぼ同様.
       See: instanceKlass::method_at_itable())
      ---------------------------------------- -}

	  // Compute start of first itableOffsetEntry (which is at the end of the vtable)

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	  int vtable_base = instanceKlass::vtable_start_offset() * wordSize;
	  int itentry_off = itableMethodEntry::method_offset_in_bytes();
	  int scan_step   = itableOffsetEntry::size() * wordSize;
	  int vte_size    = vtableEntry::size() * wordSize;
	  Address::ScaleFactor times_vte_scale = Address::times_ptr;

    {- -------------------------------------------
  (1.1) (assert)
        ---------------------------------------- -}

	  assert(vte_size == wordSize, "else adjust times_vte_scale");
	
    {- -------------------------------------------
  (1.1) コード生成:
        「クラスオブジェクト内にある itable の先頭アドレスを計算し, Rscratch に入れる」
  
         (なお, この処理は instanceKlass::start_of_itable() とほぼ同様.
          See: instanceKlass::start_of_itable())
        ---------------------------------------- -}

	  movl(scan_temp, Address(recv_klass, instanceKlass::vtable_length_offset() * wordSize));
	
	  // %%% Could store the aligned, prescaled offset in the klassoop.
	  lea(scan_temp, Address(recv_klass, scan_temp, times_vte_scale, vtable_base));
	  if (HeapWordsPerLong > 1) {
	    // Round up to align_object_offset boundary
	    // see code for instanceKlass::start_of_itable!
	    round_to(scan_temp, BytesPerLong);
	  }
	
    {- -------------------------------------------
  (1.1) コード生成:
        「index が示すオフセット分を, 先に recv_klass に足し込んでおく」
  
        (こうすると itable_index 内の情報は消えても問題なくなる. 
         itable_index がレジスタの場合は, 結果も同じレジスタに詰めることになるので (上の assert 参照)
         先に dead にしておく必要がある)
        ---------------------------------------- -}

	  // Adjust recv_klass by scaled itable_index, so we can free itable_index.
	  assert(itableMethodEntry::size() * wordSize == wordSize, "adjust the scaling in the code below");
	  lea(recv_klass, Address(recv_klass, itable_index, Address::times_ptr, itentry_off));
	
    {- -------------------------------------------
  (1.1) (以降で作るコードは以下のような処理を行う)
        ---------------------------------------- -}

	  // for (scan = klass->itable(); scan->interface() != NULL; scan += scan_step) {
	  //   if (scan->interface() == intf) {
	  //     result = (klass + scan->offset() + itable_index);
	  //   }
	  // }

    {- -------------------------------------------
  (1.1) (変数)
        ---------------------------------------- -}

	  Label search, found_method;
	
    {- -------------------------------------------
  (1.1) コード生成:
        「itable の中を先頭から順に調べていき, 
         _interface フィールドの値が目的のインターフェースに等しい要素を探す.
  
         なお, 探している途中で _interface フィールドが NULL の要素に出くわした場合は
         引数で指定された L_no_such_interface ラベルにジャンプする.」
  
  
         (ただし, ここではこの探索のループ処理を複製して ループ1.5周分だけ生成している.
          (回数は以下の peel で指定. ２回目の途中で break する).
          このため実際に生成されるコードは以下のような感じになる.
  
                          method_result = scan_temp._interface;
                          if (method_result == intf_klass) goto found_method;
                          do {
                            if (method_result == NULL) goto L_no_such_interface;
                            scan_temp++;
                            method_result = scan_temp._interface;
                          } while (method_result != intf_klass)
          found_method:   ...                                                  )
        ---------------------------------------- -}

	  for (int peel = 1; peel >= 0; peel--) {

    {- -------------------------------------------
  (1.1) (処理対象の itableOffsetEntry から _interface フィールドをロード)
        ---------------------------------------- -}

	    movptr(method_result, Address(scan_temp, itableOffsetEntry::interface_offset_in_bytes()));

    {- -------------------------------------------
  (1.1) (ロード結果を探しているインターフェースと比較. 
         1 回目は, 等しければ found_method ラベルに飛ぶ.
         2 回目は, 等しくなければ search ラベルに飛ぶ.)
        ---------------------------------------- -}

	    cmpptr(intf_klass, method_result);
	
	    if (peel) {
	      jccb(Assembler::equal, found_method);
	    } else {
	      jccb(Assembler::notEqual, search);
	      // (invert the test to fall through to found_method...)
	    }
	
	    if (!peel)  break;
	
	    bind(search);
	
    {- -------------------------------------------
  (1.1) (ロード結果が NULL だったら, L_no_such_interface にジャンプ)
        ---------------------------------------- -}

	    // Check that the previous entry is non-null.  A null entry means that
	    // the receiver class doesn't implement the interface, and wasn't the
	    // same as when the caller was compiled.
	    testptr(method_result, method_result);
	    jcc(Assembler::zero, L_no_such_interface);

    {- -------------------------------------------
  (1.1) (処理対象を次の要素に移す)
        ---------------------------------------- -}

	    addptr(scan_temp, scan_step);
	  }
	
	  bind(found_method);

  {- -------------------------------------------
  (1) (対応する itableOffsetEntry を探す処理はここまで)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) コード生成:
      「探し当てた itableOffsetEntry 内の offset 値から適切な vtable の位置を求め, 
        さらにその結果と (index 分を足し込んだ)クラスオブジェクトのアドレスを足すことで
        呼び出し先の methodOop を取得する.
        取得した結果は, 引数の method_result で指定された レジスタに格納.」
      ---------------------------------------- -}

	  // Got a hit.
	  movl(scan_temp, Address(scan_temp, itableOffsetEntry::offset_offset_in_bytes()));
	  movptr(method_result, Address(recv_klass, scan_temp, Address::times_1));
	}
	
```


