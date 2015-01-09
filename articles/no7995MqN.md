---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/sharedRuntime.cpp
### 説明(description)

```
// Create a native wrapper for this native method.  The wrapper converts the
// java compiled calling convention to the native convention, handlizes
// arguments, and transitions to native.  On return from the native we transition
// back to java blocking if a safepoint is in progress.
```

### 名前(function name)
```
nmethod *AdapterHandlerLibrary::create_native_wrapper(methodHandle method, int compile_id) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ResourceMark)
      ---------------------------------------- -}

	  ResourceMark rm;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  nmethod* nm = NULL;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(method->has_native_function(), "must have something valid to call!");
	
  {- -------------------------------------------
  (1) (以下の処理は AdapterHandlerLibrary_lock で排他した状態で行う)
      ---------------------------------------- -}

	  {
	    // perform the work while holding the lock, but perform any printing outside the lock
	    MutexLocker mu(AdapterHandlerLibrary_lock);

  {- -------------------------------------------
  (1) もし誰かが入れ違いで JIT を完了してしまっていたら, それを使うだけでいい (= もう JIT コンパイルを行う必要は無い) 
      ここでリターン.
      ---------------------------------------- -}

	    // See if somebody beat us to it
	    nm = method->code();
	    if (nm) {
	      return nm;
	    }
	
  {- -------------------------------------------
  (1) (ResourceMark)
      ---------------------------------------- -}

	    ResourceMark rm;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (JIT コンパイル処理で使用するデータ構造の準備)
      ---------------------------------------- -}

	    BufferBlob*  buf = buffer_blob(); // the temporary code buffer in CodeCache
	    if (buf != NULL) {
	      CodeBuffer buffer(buf);
	      double locs_buf[20];
	      buffer.insts()->initialize_shared_locs((relocInfo*)locs_buf, sizeof(locs_buf) / sizeof(relocInfo));
	      MacroAssembler _masm(&buffer);
	
	      // Fill in the signature array, for the calling-convention call.
	      int total_args_passed = method->size_of_parameters();
	
	      BasicType* sig_bt = NEW_RESOURCE_ARRAY(BasicType,total_args_passed);
	      VMRegPair*   regs = NEW_RESOURCE_ARRAY(VMRegPair,total_args_passed);
	      int i=0;
	      if( !method->is_static() )  // Pass in receiver first
	        sig_bt[i++] = T_OBJECT;
	      SignatureStream ss(method->signature());
	      for( ; !ss.at_return_type(); ss.next()) {
	        sig_bt[i++] = ss.type();  // Collect remaining bits of signature
	        if( ss.type() == T_LONG || ss.type() == T_DOUBLE )
	          sig_bt[i++] = T_VOID;   // Longs & doubles take 2 Java slots
	      }
	      assert( i==total_args_passed, "" );
	      BasicType ret_type = ss.type();
	
  {- -------------------------------------------
  (1) SharedRuntime::java_calling_convention() を呼んで, 
      JIT 生成するコードの calling convention 及びレジスタに乗らない引数の個数を算出しておく.
      ---------------------------------------- -}

	      // Now get the compiled-Java layout as input arguments
	      int comp_args_on_stack;
	      comp_args_on_stack = SharedRuntime::java_calling_convention(sig_bt, regs, total_args_passed, false);
	
  {- -------------------------------------------
  (1) SharedRuntime::generate_native_wrapper() を呼んで, JIT コンパイル処理を行う.
      ---------------------------------------- -}

	      // Generate the compiled-to-native wrapper code
	      nm = SharedRuntime::generate_native_wrapper(&_masm,
	                                                  method,
	                                                  compile_id,
	                                                  total_args_passed,
	                                                  comp_args_on_stack,
	                                                  sig_bt,regs,
	                                                  ret_type);
	    }

  {- -------------------------------------------
  (1) (ここまでが AdapterHandlerLibrary_lock で排他した処理)
      ---------------------------------------- -}

	  }
	
	  // Must unlock before calling set_code
	
  {- -------------------------------------------
  (1) JIT コンパイルが成功していれば, methodOopDesc::set_code() を呼び出して, 
      処理対象の methodOopDesc 中の _from_compiled_entry や _from_interpreted_entry を
      JIT 生成したコードに置き換える.
  
      (なお, JIT コンパイルが成功していた場合は, 
       nmethod::post_compiled_method_load_event() を呼んで 
       JVMTI のイベント通知も行っている ("CompiledMethodLoad")
       (See: [here](no3718UPQ.html) for details))
  
      (逆に JIT コンパイルが失敗していた場合は, CodeCache に空きがない可能性が高いので(?), 
      CompileBroker::handle_full_code_cache() を呼んで
      これ以上の JIT コンパイルを禁止するかあるいは空き領域を捻出する処理を行っている)
      ---------------------------------------- -}

	  // Install the generated code.
	  if (nm != NULL) {
	    if (PrintCompilation) {
	      ttyLocker ttyl;
	      CompileTask::print_compilation(tty, nm, method->is_static() ? "(static)" : "");
	    }
	    method->set_code(method, nm);
	    nm->post_compiled_method_load_event();
	  } else {
	    // CodeCache is full, disable compilation
	    CompileBroker::handle_full_code_cache();
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return nm;
	}
	
```


