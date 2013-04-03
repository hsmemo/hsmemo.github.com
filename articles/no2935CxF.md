---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/assembler_sparc.cpp

### 名前(function name)
```
void MacroAssembler::g1_write_barrier_pre(Register obj,
                                          Register index,
                                          int offset,
                                          Register pre_val,
                                          Register tmp,
                                          bool preserve_o_regs) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label filtered;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  if (obj == noreg) {
	    // We are not loading the previous value so make
	    // sure that we don't trash the value in pre_val
	    // with the code below.
	    assert_different_registers(pre_val, tmp);
	  } else {
	    // We will be loading the previous value
	    // in this code so...
	    assert(offset == 0 || index == noreg, "choose one");
	    assert(pre_val == noreg, "check this code");
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「JavaThread::_satb_mark_queue フィールドの ObjPtrQueue オブジェクト内の
        _active フィールドをロードする」
      
      (このフィールドは, write barrier でポインタの記録が必要かどうかを示す.
       ConcurrentMarkThread が動作中であれば true になる.
       See: SATBMarkQueueSet::set_active_all_threads())
  					   
      (なおコンパイラによって bool のバイト数が異なるので, 
       バイト数に応じて２通りのコードを生成する.
       ここでは 4byte になる場合と 1byte になる場合を想定している模様)
      ---------------------------------------- -}

	  // Is marking active?
	  if (in_bytes(PtrQueue::byte_width_of_active()) == 4) {
	    ld(G2,
	       in_bytes(JavaThread::satb_mark_queue_offset() +
	                PtrQueue::byte_offset_of_active()),
	       tmp);
	  } else {
	    guarantee(in_bytes(PtrQueue::byte_width_of_active()) == 1,
	              "Assumption");
	    ldsb(G2,
	         in_bytes(JavaThread::satb_mark_queue_offset() +
	                  PtrQueue::byte_offset_of_active()),
	         tmp);
	  }
	
  {- -------------------------------------------
  (1) コード生成:
      「もし _active フィールドの値が 0 (= false) であれば,
       何もする必要が無いので, filtered ラベルまでジャンプ」
      ---------------------------------------- -}

	  // Check on whether to annul.
	  br_on_reg_cond(rc_z, /*annul*/false, Assembler::pt, tmp, filtered);
	  delayed() -> nop();
	
  {- -------------------------------------------
  (1) コード生成:
      「書き換え対象箇所の現在の値をロードする」
      ---------------------------------------- -}

	  // Do we need to load the previous value?
	  if (obj != noreg) {
	    // Load the previous value...
	    if (index == noreg) {
	      if (Assembler::is_simm13(offset)) {
	        load_heap_oop(obj, offset, tmp);
	      } else {
	        set(offset, tmp);
	        load_heap_oop(obj, tmp, tmp);
	      }
	    } else {
	      load_heap_oop(obj, index, tmp);
	    }
	    // Previous value has been loaded into tmp
	    pre_val = tmp;
	  }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(pre_val != noreg, "must have a real register");
	
  {- -------------------------------------------
  (1) コード生成:
      「もし現在の値が NULL であれば, 記録する価値はないので, filtered ラベルまでジャンプ」
      ---------------------------------------- -}

	  // Is the previous value null?
	  // Check on whether to annul.
	  br_on_reg_cond(rc_z, /*annul*/false, Assembler::pt, pre_val, filtered);
	  delayed() -> nop();
	
  {- -------------------------------------------
  (1) (ここまで到達したケースは enqueue を行う必要があるケース.
       なお, 大抵は pre_val はスクラッチ用途の G レジスタになっているが, 稀に O レジスタになることもある.
       G レジスタの場合は普通に call 命令を使えばいい.
       O レジスタの場合は先に save してから call する.)
      ---------------------------------------- -}

	  // OK, it's not filtered, so we'll need to call enqueue.  In the normal
	  // case, pre_val will be a scratch G-reg, but there are some cases in
	  // which it's an O-reg.  In the first case, do a normal call.  In the
	  // latter, do a save here and call the frameless version.
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  guarantee(pre_val->is_global() || pre_val->is_out(),
	            "Or we need to think harder.");
	
  {- -------------------------------------------
  (1) 以下の if ブロック内で実際の記録処理を行うコードを生成する. 生成されるコードは以下の2通り.
      (なお, これらの中で使用される satb_log_enqueue_with_frame 大域変数, 及び
       satb_log_enqueue_with_frame 大域変数は, 遅延初期化される.
       遅延初期化処理は, 直前で呼び出している generate_satb_log_enqueue_if_necessary() が行う.)
    
      * 現在の値をロードしたレジスタ(pre_val)が G レジスタで, 
        かつ O レジスタの値を保存しなくてもよい(= preserve_o_regs 引数が false)場合:
    
        コード生成:
        「call 命令で satb_log_enqueue_with_frame 大域変数が指すアドレスにジャンプする.
         (ついでに, 遅延スロットで pre_val の値を O0 レジスタにコピーしておく)」
  
      * それ以外の場合:
  
        コード生成:
        「call 命令で satb_log_enqueue_frameless 大域変数が指すアドレスにジャンプする.
         呼び出しの前後では save/restore も行う.
         (ついでに, 遅延スロットで pre_val の値を O0 レジスタにコピーしておく)」
      ---------------------------------------- -}

	  if (pre_val->is_global() && !preserve_o_regs) {
	    generate_satb_log_enqueue_if_necessary(true); // with frame
	
	    call(satb_log_enqueue_with_frame);
	    delayed()->mov(pre_val, O0);
	  } else {
	    generate_satb_log_enqueue_if_necessary(false); // frameless
	
	    save_frame(0);
	    call(satb_log_enqueue_frameless);
	    delayed()->mov(pre_val->after_save(), O0);
	    restore();
	  }
	
  {- -------------------------------------------
  (1) (ここが filtered ラベルの位置)
      ---------------------------------------- -}

	  bind(filtered);
	}
	
```


