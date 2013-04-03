---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp

### 名前(function name)
```
instanceOop Management::create_thread_info_instance(ThreadSnapshot* snapshot,
                                                    objArrayHandle monitors_array,
                                                    typeArrayHandle depths_array,
                                                    objArrayHandle synchronizers_array,
                                                    TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) instanceKlassHandle::allocate_instance_handle() で
      新しい java.lang.management.ThreadInfo オブジェクトを生成し, 
      さらに JavaCalls::call_special() でコンストラクタを呼び出して初期化も行う.
    
        (なお, コンストラクタとしては, 
         引数スロットが 17 個文のコンストラクタ(= ロック情報付きのコンストラクタ)が呼ばれる)
      
        (なお, コンストラクタに渡す引数のほとんどは, 
         initialize_ThreadInfo_constructor_arguments() という補助関数内でセットされている)
  
      返値としては, 生成した java.lang.management.ThreadInfo オブジェクトが返される.
      ---------------------------------------- -}

	  klassOop k = Management::java_lang_management_ThreadInfo_klass(CHECK_NULL);
	  instanceKlassHandle ik (THREAD, k);
	
	  JavaValue result(T_VOID);
	  JavaCallArguments args(17);
	
	  // First allocate a ThreadObj object and
	  // push the receiver as the first argument
	  Handle element = ik->allocate_instance_handle(CHECK_NULL);
	  args.push_oop(element);
	
	  // initialize the arguments for the ThreadInfo constructor
	  initialize_ThreadInfo_constructor_arguments(&args, snapshot, CHECK_NULL);
	
	  // push the locked monitors and synchronizers in the arguments
	  args.push_oop(monitors_array);
	  args.push_oop(depths_array);
	  args.push_oop(synchronizers_array);
	
	  // Call ThreadInfo constructor with locked monitors and synchronizers
	  JavaCalls::call_special(&result,
	                          ik,
	                          vmSymbols::object_initializer_name(),
	                          vmSymbols::java_lang_management_ThreadInfo_with_locks_constructor_signature(),
	                          &args,
	                          CHECK_NULL);
	
	  return (instanceOop) element();
	}
	
```


