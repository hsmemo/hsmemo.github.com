---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
IRT_ENTRY(void, InterpreterRuntime::at_safepoint(JavaThread* thread))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (コメントによると, 
       昔は invoke 系のバイトコード用に引数を待避する処理をここで行っていたが, 今では必要なくなった, 
       とのこと)
      ---------------------------------------- -}

	  // We used to need an explict preserve_arguments here for invoke bytecodes. However,
	  // stack traversal automatically takes care of preserving arguments for invoke, so
	  // this is no longer needed.
	
  {- -------------------------------------------
  (1) (コメントによると, 
       IRT_END で暗黙的に safepoint チェックが行われるので, 
       この関数が safepoint 中に呼ばれた場合はブロックされることが保証されている, 
       とのこと.)
      ---------------------------------------- -}

	  // IRT_END does an implicit safepoint check, hence we are guaranteed to block
	  // if this is called during a safepoint
	
  {- -------------------------------------------
  (1) JvmtiExport::should_post_single_step() が true であれば, 
      JvmtiExport::at_single_stepping_point() を呼び出す.
      (See: [here](no7882EDP.html) for details)
      ---------------------------------------- -}

	  if (JvmtiExport::should_post_single_step()) {
	    // We are called during regular safepoints and when the VM is
	    // single stepping. If any thread is marked for single stepping,
	    // then we may have JVMTI work to do.
	    JvmtiExport::at_single_stepping_point(thread, method(thread), bcp(thread));
	  }

  {- -------------------------------------------
  (1) この IRT_END 時に Safepoint チェックが入る.
      (正確には, IRT_END というよりは, IRT_ENTRY 内で宣言された ThreadInVMfromJava のデストラクタ処理)
  
      (JvmtiExport::should_post_single_step() が true でなければ, dispatch table は元に戻される.
       JvmtiExport::should_post_single_step() が true であれば, dispatch table は変更されない.
       See: TemplateInterpreter::ignore_safepoints())
      ---------------------------------------- -}

	IRT_END
	
```


