---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/shark/sharkCompiler.cpp

### 名前(function name)
```
nmethod* SharkCompiler::generate_native_wrapper(MacroAssembler* masm,
                                                methodHandle    target,
                                                int             compile_id,
                                                BasicType*      arg_types,
                                                BasicType       return_type) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(is_initialized(), "should be");

  {- -------------------------------------------
  (1) (ResourceMark)
      ---------------------------------------- -}

	  ResourceMark rm;

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const char *name = methodname(
	    target->klass_name()->as_utf8(), target->name()->as_utf8());
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Create the code buffer and builder
	  SharkCodeBuffer cb(masm);
	  SharkBuilder builder(&cb);
	
	  // Emit the entry point
	  SharkEntry *entry = (SharkEntry *) cb.malloc(sizeof(SharkEntry));
	
  {- -------------------------------------------
  (1) 生成するスタブの内容を表現した LLVM IR を作成する.
      ---------------------------------------- -}

	  // Build the LLVM IR for the method
	  SharkNativeWrapper *wrapper = SharkNativeWrapper::build(
	    &builder, target, name, arg_types, return_type);
	
  {- -------------------------------------------
  (1) SharkCompiler::generate_native_code() を呼んで, 
      作成した LLVM IR からマシン語コードを生成する.
      ---------------------------------------- -}

	  // Generate native code
	  generate_native_code(entry, wrapper->function(), name);
	
  {- -------------------------------------------
  (1) nmethod::new_native_nmethod() を呼んで, できたコードを登録する
      ---------------------------------------- -}

	  // Return the nmethod for installation in the VM
	  return nmethod::new_native_nmethod(target,
	                                     compile_id,
	                                     masm->code(),
	                                     0,
	                                     0,
	                                     wrapper->frame_size(),
	                                     wrapper->receiver_offset(),
	                                     wrapper->lock_offset(),
	                                     wrapper->oop_maps());
	}
	
```


