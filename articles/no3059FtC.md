---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/interpreterRT_x86_64.cpp

### 名前(function name)
```
IRT_ENTRY(address,
          InterpreterRuntime::slow_signature_handler(JavaThread* thread,
                                                     methodOopDesc* method,
                                                     intptr_t* from,
                                                     intptr_t* to))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodHandle m(thread, (methodOop)method);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(m->is_native(), "sanity check");
	
  {- -------------------------------------------
  (1) この場で新たに生成した SlowSignatureHandler オブジェクトに対して 
      SignatureIterator::iterate() を呼び出すことにより, 引数を
      from の位置(インタープリタのフレーム内)から 
      to の位置(の native の ABI に対応した位置)へとコピーする.
      ---------------------------------------- -}

	  // handle arguments
	  SlowSignatureHandler(m, (address)from, to + 1).iterate(UCONST64(-1));
	
  {- -------------------------------------------
  (1) Interpreter::result_handler() を呼んで返値の型に合った result handler を取得し, 
      それをリターン.
      ---------------------------------------- -}

	  // return result handler
	  return Interpreter::result_handler(m->result_type());
	IRT_END
	
```


