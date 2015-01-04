---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/generateOopMap.cpp

### 名前(function name)
```
void GenerateOopMap::compute_map(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      ---------------------------------------- -}

	#ifndef PRODUCT
	  if (TimeOopMap2) {
	    method()->print_short_name(tty);
	    tty->print("  ");
	  }
	  if (TimeOopMap) {
	    _total_byte_count += method()->code_size();
	  }
	#endif

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  TraceTime t_single("oopmap time", TimeOopMap2);
	  TraceTime t_all(NULL, &_total_oopmap_time, TimeOopMap);
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // Initialize values
	  _got_error      = false;
	  _conflict       = false;
	  _max_locals     = method()->max_locals();
	  _max_stack      = method()->max_stack();
	  _has_exceptions = (method()->exception_table()->length() > 0);
	  _nof_refval_conflicts = 0;
	  _init_vars      = new GrowableArray<intptr_t>(5);  // There are seldom more than 5 init_vars
	  _report_result  = false;
	  _report_result_for_send = false;
	  _new_var_map    = NULL;
	  _ret_adr_tos    = new GrowableArray<intptr_t>(5);  // 5 seems like a good number;
	  _did_rewriting  = false;
	  _did_relocation = false;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceNewOopMapGeneration) {
	    tty->print("Method name: %s\n", method()->name()->as_C_string());
	    if (Verbose) {
	      _method->print_codes();
	      tty->print_cr("Exception table:");
	      typeArrayOop excps = method()->exception_table();
	      for(int i = 0; i < excps->length(); i += 4) {
	        tty->print_cr("[%d - %d] -> %d", excps->int_at(i + 0), excps->int_at(i + 1), excps->int_at(i + 2));
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 以下の条件のどちらかを満たす場合は
      計算するまでもなく OopMap は空だと分かるので, ここでリターンする.
       
      * コードが空っぽ
      * 局所変数サイズもオペランドスタックサイズも 0
  
      (念のために, リターン前に fill_stackmap_prolog() と fill_stackmap_epilog() を呼んでいる.
       ただし, 現状ではどのサブクラスでも何もしていない模様)
      ---------------------------------------- -}

	  // if no code - do nothing
	  // compiler needs info
	  if (method()->code_size() == 0 || _max_locals + method()->max_stack() == 0) {
	    fill_stackmap_prolog(0);
	    fill_stackmap_epilog();
	    return;
	  }

  {- -------------------------------------------
  (1) 以下の順でバイトコードの抽象実行処理を行う.
  
      (1) RetTable::compute_ret_table() を呼んで, 
          全ての jsr 命令の情報を収集する.
  
      (2) GenerateOopMap::mark_bbheaders_and_count_gc_points() を呼んで, 
          全ての basic block (の先頭の位置)を計算する
          (ついでに safepoint になり得る箇所の個数も数えているようだが, こちらの仕組みは現状では使われていないような... #TODO)
  
      (3) GenerateOopMap::do_interpretation() を呼んで, 
          バイトコードの抽象実行を行う.
          (これにより, 各 BasicBlock オブジェクトに, その先頭時点での状態が記録される.
           また, モニターが balance しているかどうか等の情報も収集される)
          
      (4) GenerateOopMap::report_result() を呼んで, 
          全てのバイトコード命令に対して 
          GenerateOopMap::fill_stackmap_for_opcodes() (を各サブクラスがオーバーライドしたもの) を呼び出す.
  
          (ただしこの(4)の処理については, この処理を必要とするサブクラスでしか実行されない.
           より具体的に言うと, GenerateOopMap::report_results() (を各サブクラスがオーバーライドしたもの) が
           true を返す場合にしか実行されない.)
  
      (ただし, それぞれの処理を行う時点で
       何かエラーが起こっていないか (= _got_error フィールドが true でないか) をチェックし, 
       起こっていたらそれ以降の処理は省略する (See: GenerateOopMap::report_error(), GenerateOopMap::verify_error()).
       この場合は, さらに, この関数の最後で例外が送出される.)
      ---------------------------------------- -}

	  // Step 1: Compute all jump targets and their return value
	  if (!_got_error)
	    _rt.compute_ret_table(_method);
	
	  // Step 2: Find all basic blocks and count GC points
	  if (!_got_error)
	    mark_bbheaders_and_count_gc_points();
	
	  // Step 3: Calculate stack maps
	  if (!_got_error)
	    do_interpretation();
	
	  // Step 4:Return results
	  if (!_got_error && report_results())
	     report_result();
	
	  if (_got_error) {
	    THROW_HANDLE(_exception);
	  }
	}
	
```


