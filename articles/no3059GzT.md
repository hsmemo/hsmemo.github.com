---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreter_x86_64.cpp

### 名前(function name)
```
address InterpreterGenerator::generate_math_entry(AbstractInterpreter::MethodKind kind) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (生成したコードが実行される時点では, レジスタには以下の値が入っているはず)
      ---------------------------------------- -}

	  // rbx,: methodOop
	  // rcx: scratrch
	  // r13: sender sp
	
  {- -------------------------------------------
  (1) InlineIntrinsics オプションが指定されていない場合には, 特にすることはない.
      単に NULL をリターンするだけ.
      ---------------------------------------- -}

	  if (!InlineIntrinsics) return NULL; // Generate a vanilla entry
	
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry_point = __ pc();
	
  {- -------------------------------------------
  (1) (以下の箇所では (今のところ) Safepoint のチェックを行う必要はないとのこと)
      ---------------------------------------- -}

	  // These don't need a safepoint check because they aren't virtually
	  // callable. We won't enter these intrinsics from compiled code.
	  // If in the future we added an intrinsic which was virtually callable
	  // we'd have to worry about how to safepoint so that this code is used.
	
  {- -------------------------------------------
  (1) (以下のコードは, JIT コンパイラのコードと同じにしている.
       インタープリタからコンパイラに切り替わった際に変なバグを生まないように
       コードは同一にしておくべき, とのこと)
      ---------------------------------------- -}

	  // mathematical functions inlined by compiler
	  // (interpreter must provide identical implementation
	  // in order to avoid monotonicity bugs when switching
	  // from interpreter to compiler in the middle of some
	  // computation)
	  //
	  // stack: [ ret adr ] <-- rsp
	  //        [ lo(arg) ]
	  //        [ hi(arg) ]
	  //
	
  {- -------------------------------------------
  (1) JDK 1.2 の頃は StrictMath がなかったため
      sin()/cos()/sqrt() 等をネイティブメソッドで実装していたが, 
      現在では JDK 1.2 は EOL になったためその心配は無いとのこと.    
      ---------------------------------------- -}

	  // Note: For JDK 1.2 StrictMath doesn't exist and Math.sin/cos/sqrt are
	  //       native methods. Interpreter::method_kind(...) does a check for
	  //       native methods first before checking for intrinsic methods and
	  //       thus will never select this entry point. Make sure it is not
	  //       called accidentally since the SharedRuntime entry points will
	  //       not work for JDK 1.2.
	  //
	  // We no longer need to check for JDK 1.2 since it's EOL'ed.
	  // The following check existed in pre 1.6 implementation,
	  //    if (Universe::is_jdk12x_version()) {
	  //      __ should_not_reach_here();
	  //    }
	  // Universe::is_jdk12x_version() always returns false since
	  // the JDK version is not yet determined when this method is called.
	  // This method is called during interpreter_init() whereas
	  // JDK version is only determined when universe2_init() is called.
	
	  // Note: For JDK 1.3 StrictMath exists and Math.sin/cos/sqrt are
	  //       java methods.  Interpreter::method_kind(...) will select
	  //       this entry point for the corresponding methods in JDK 1.3.
	  // get argument
	
  {- -------------------------------------------
  (1) コード生成:
      (引数で指定された種別の算術演算を行うコードを生成する)
      ---------------------------------------- -}

	  if (kind == Interpreter::java_lang_math_sqrt) {
	    __ sqrtsd(xmm0, Address(rsp, wordSize));
	  } else {
	    __ fld_d(Address(rsp, wordSize));
	    switch (kind) {
	      case Interpreter::java_lang_math_sin :
	          __ trigfunc('s');
	          break;
	      case Interpreter::java_lang_math_cos :
	          __ trigfunc('c');
	          break;
	      case Interpreter::java_lang_math_tan :
	          __ trigfunc('t');
	          break;
	      case Interpreter::java_lang_math_abs:
	          __ fabs();
	          break;
	      case Interpreter::java_lang_math_log:
	          __ flog();
	          break;
	      case Interpreter::java_lang_math_log10:
	          __ flog10();
	          break;
	      default                              :
	          ShouldNotReachHere();
	    }
	
	    // return double result in xmm0 for interpreter and compilers.
	    __ subptr(rsp, 2*wordSize);
	    // Round to 64bit precision
	    __ fstp_d(Address(rsp, 0));
	    __ movdbl(xmm0, Address(rsp, 0));
	    __ addptr(rsp, 2*wordSize);
	  }
	
	
  {- -------------------------------------------
  (1) コード生成:
      「SP を r13 の値に復帰させた後, リターン」
      ---------------------------------------- -}

	  __ pop(rax);
	  __ mov(rsp, r13);
	  __ jmp(rax);
	
  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry_point;
	}
	
```


