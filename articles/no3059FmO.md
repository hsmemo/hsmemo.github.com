---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)

```
// A result handler converts/unboxes a native call result into
// a java interpreter/compiler result. The current frame is an
// interpreter frame. The activation frame unwind code must be
// consistent with that of TemplateTable::_return(...). In the
// case of native methods, the caller's SP was not modified.
```

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_result_handler_for(BasicType type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Itos_i  = Otos_i ->after_save();
	  Register Itos_l  = Otos_l ->after_save();
	  Register Itos_l1 = Otos_l1->after_save();
	  Register Itos_l2 = Otos_l2->after_save();

  {- -------------------------------------------
  (1) コード生成:
      引数で指定された型に応じて, 適切なコードを生成する.
      (ここの処理でやることは, 値の bit 幅をそろえるのと, 
       レジスタウィンドウを回した後の TOS レジスタに値を移動させるのが目的)
  
      * T_BOOLEAN
       「O0 が非ゼロなら I0(= Itos_i) に 1 をセットする. 0 なら I0(= Itos_i) に 0 をセットする.
         (subcc で G0 から引き算する(非ゼロなら carry が出る). その後 addc で G0 と 0 と carry を足せばいい)」
      * T_CHAR   
       「O0 の下位 16bit(符号無し) 分だけを I0(= Itos_i) にコピー.
         (sll で 16bit 左シフトした後, srl で上位 32bit を 0 クリアしながら 16bit 論理右シフト)」
      * T_BYTE   
       「O0 の下位 8bit(符号あり) 分だけを I0(= Itos_i) にコピー.
         (sll で 24bit 左シフトした後, sra で上位 32bit を 0 クリアしながら 24bit 算術右シフト)」
      * T_SHORT  
       「O0 の下位 16bit(符号あり) 分だけを I0(= Itos_i) にコピー.
         (sll で 16bit 左シフトした後, sra で上位 32bit を 0 クリアしながら 16bit 算術右シフト)」
      * T_LONG   
        (以下の T_INT の処理にフォールスルー. ただし 32bit 版だとその前に O1 を I1(= Itos_l2) にコピーしている)
      * T_INT    
       「O0 の値を I0(= Itos_i) にコピー.」
      * T_VOID   
       「何もしない」
      * T_FLOAT  
       「何もしない (F0 と Ftos_f は同じはずだし, レジスタウィンドウを回しても
         浮動小数レジスタには影響しないので, 何もしなくていい)」
      * T_DOUBLE 
       「何もしない (F0 と Ftos_d は同じはずだし, レジスタウィンドウを回しても
         浮動小数レジスタには影響しないので, 何もしなくていい)」
      * T_OBJECT 
       「フレーム中の frame::interpreter_frame_oop_temp_offset 野市に待避していた値を I0(= Itos_i) にロードする.」
      ---------------------------------------- -}

	  switch (type) {
	    case T_BOOLEAN: __ subcc(G0, O0, G0); __ addc(G0, 0, Itos_i); break; // !0 => true; 0 => false
	    case T_CHAR   : __ sll(O0, 16, O0); __ srl(O0, 16, Itos_i);   break; // cannot use and3, 0xFFFF too big as immediate value!
	    case T_BYTE   : __ sll(O0, 24, O0); __ sra(O0, 24, Itos_i);   break;
	    case T_SHORT  : __ sll(O0, 16, O0); __ sra(O0, 16, Itos_i);   break;
	    case T_LONG   :
	#ifndef _LP64
	                    __ mov(O1, Itos_l2);  // move other half of long
	#endif              // ifdef or no ifdef, fall through to the T_INT case
	    case T_INT    : __ mov(O0, Itos_i);                         break;
	    case T_VOID   : /* nothing to do */                         break;
	    case T_FLOAT  : assert(F0 == Ftos_f, "fix this code" );     break;
	    case T_DOUBLE : assert(F0 == Ftos_d, "fix this code" );     break;
	    case T_OBJECT :
	      __ ld_ptr(FP, (frame::interpreter_frame_oop_temp_offset*wordSize) + STACK_BIAS, Itos_i);
	      __ verify_oop(Itos_i);
	      break;
	    default       : ShouldNotReachHere();
	  }

  {- -------------------------------------------
  (1) コード生成:
      「restore 命令でレジスタを復帰(SP も I5_savedSP に復帰)させつつ, リターンする」
      ---------------------------------------- -}

	  __ ret();                           // return from interpreter activation
	  __ delayed()->restore(I5_savedSP, G0, SP);  // remove interpreter frame

  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	  NOT_PRODUCT(__ emit_long(0);)       // marker for disassembly

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


