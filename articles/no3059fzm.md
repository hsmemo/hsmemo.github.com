---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interpreterRT_sparc.cpp

### 名前(function name)
```
void InterpreterRuntime::SignatureHandlerGenerator::pass_double() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Argument  jni_arg(jni_offset(), false);

  {- -------------------------------------------
  (1) コード生成: (なお, こちらは 64bit 環境の場合のコード)
      「まずインタープリタのフレーム内に置かれている引数を Rtmp にロードし, 
        次に MacroAssembler::store_float_argument() が生成するコードで
        ロードした値を native の ABI での適切な位置にコピーする.
  
        (なお, ロードする際のフレーム内の位置は Interpreter::local_offset_in_bytes() で計算)」
      ---------------------------------------- -}

	#ifdef _LP64
	  FloatRegister  Rtmp = F0;
	  __ ldf(FloatRegisterImpl::D, Llocals, Interpreter::local_offset_in_bytes(offset() + 1), Rtmp);
	  __ store_double_argument(Rtmp, jni_arg);

  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	#else
	  Register  Rtmp = O0;
	  __ ld(Llocals, Interpreter::local_offset_in_bytes(offset() + 1), Rtmp);
	  __ store_argument(Rtmp, jni_arg);
	  __ ld(Llocals, Interpreter::local_offset_in_bytes(offset()), Rtmp);
	  Argument successor(jni_arg.successor());
	  __ store_argument(Rtmp, successor);
	#endif
	}
	
```


