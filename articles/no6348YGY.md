---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/compilationPolicy.cpp

### 名前(function name)
```
nmethod* NonTieredCompPolicy::event(methodHandle method, methodHandle inlinee, int branch_bci, int bci, CompLevel comp_level, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(comp_level == CompLevel_none, "This should be only called from the interpreter");

  {- -------------------------------------------
  (1) (デバッグ用の処理) (NOT_PRODUCT 時にのみ実行)
      ---------------------------------------- -}

	  NOT_PRODUCT(trace_frequency_counter_overflow(method, branch_bci, bci));

  {- -------------------------------------------
  (1) JVMTI の処理のために interp_only_mode になっている場合, 
      計測処理(& CompilerBroker の呼び出し)は行わずに, ここでリターン (返値は NULL).
      (See: [here](no3059eFS.html) for details)
      ---------------------------------------- -}

	  if (JvmtiExport::can_post_interpreter_events()) {
	    assert(THREAD->is_Java_thread(), "Wrong type of thread");
	    if (((JavaThread*)THREAD)->is_interp_only_mode()) {
	      // If certain JVMTI events (e.g. frame pop event) are requested then the
	      // thread is forced to remain in interpreted code. This is
	      // implemented partly by a check in the run_compiled_code
	      // section of the interpreter whether we should skip running
	      // compiled code, and partly by skipping OSR compiles for
	      // interpreted-only threads.
	      if (bci != InvocationEntryBci) {
	        reset_counter_for_back_branch_event(method);
	        return NULL;
	      }
	    }
	  }

  {- -------------------------------------------
  (1) 以下の処理は, 対象の位置がメソッドの先頭か backward branch かに応じて 2通りに分岐.
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) メソッドの先頭の場合 (= メソッド全体を JIT コンパイルする場合),
      NonTieredCompPolicy::method_invocation_event() (をサブクラスがオーバーライドしたもの) を呼び出して JIT コンパイル処理を行い,
      NULL をリターンする.
  
      (ただし, 既に JIT コンパイル済みになっている場合や UseCompiler オプションが指定されていない場合は
       JIT コンパイル処理は行わない.
       この場合は, 代わりに NonTieredCompPolicy::reset_counter_for_invocation_event() を呼んで
       (すぐにまた JIT コンパイルが発生しないように) InvocationCounter を下げ, 
       その後に NULL をリターン)
      ---------------------------------------- -}

	if (bci == InvocationEntryBci) {
	    // when code cache is full, compilation gets switched off, UseCompiler
	    // is set to false
	    if (!method->has_compiled_code() && UseCompiler) {
	      method_invocation_event(method, CHECK_NULL);
	    } else {
	      // Force counter overflow on method entry, even if no compilation
	      // happened.  (The method_invocation_event call does this also.)
	      reset_counter_for_invocation_event(method);
	    }
	    // compilation at an invocation overflow no longer goes and retries test for
	    // compiled method. We always run the loser of the race as interpreted.
	    // so return NULL
	    return NULL;

  {- -------------------------------------------
  (1) backward branch の場合 (= OnStackReplacement の場合), 
      以下のようにして JIT コンパイル結果, 又は NULL をリターンする.
      
      * (他のスレッドが JIT コンパイルリクエストを並行に出すなどして
        既に JIT コンパイル済みになっている可能性もあるので(?))
        まず methodOopDesc::lookup_osr_nmethod_for() を呼んで
        JIT コンパイル結果の取得を試みる.
      
        取得に成功すれば, それをリターン.
      
      * まだ JIT コンパイル結果がない場合, UseCompiler オプションが指定されていれば
        NonTieredCompPolicy::method_back_branch_event() (をサブクラスがオーバーライドしたもの) を呼び出して JIT コンパイル処理を行い,
        methodOopDesc::lookup_osr_nmethod_for() でコンパイル結果を取得する.
  
        取得に成功すれば, それをリターン.
  
      * 以上の処理で JIT コンパイル結果を取得できていない場合, 
        NULL をリターンする.
  
        なお, NULL をリターンする直前に NonTieredCompPolicy::reset_counter_for_back_branch_event() を呼んで
        backward branch の InvocationCounter を下げ, 代わりに
        メソッド全体の呼び出し回数を表す InvocationCounter を増加させている.
        (次回のこの backward branch での JIT コンパイル発生を遅らせる代わりに,
        メソッド全体の JIT コンパイルが起こりやすくなる)
      ---------------------------------------- -}

	  } else {
	    // counter overflow in a loop => try to do on-stack-replacement
	    nmethod* osr_nm = method->lookup_osr_nmethod_for(bci, CompLevel_highest_tier, true);
	    NOT_PRODUCT(trace_osr_request(method, osr_nm, bci));
	    // when code cache is full, we should not compile any more...
	    if (osr_nm == NULL && UseCompiler) {
	      method_back_branch_event(method, bci, CHECK_NULL);
	      osr_nm = method->lookup_osr_nmethod_for(bci, CompLevel_highest_tier, true);
	    }
	    if (osr_nm == NULL) {
	      reset_counter_for_back_branch_event(method);
	      return NULL;
	    }
	    return osr_nm;
	  }

  {- -------------------------------------------
  (1) (この return は現状到達するパスはなさそうだが, 念のため?)
      ---------------------------------------- -}

	  return NULL;
	}
	
```


