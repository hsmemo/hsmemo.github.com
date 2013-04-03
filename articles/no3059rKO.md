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
void InterpreterRuntime::SignatureHandlerGenerator::pass_word(int size_of_arg, int offset_in_arg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Argument  jni_arg(jni_offset() + offset_in_arg, false);
	  Register     Rtmp = O0;

  {- -------------------------------------------
  (1) コード生成:
      「まずインタープリタのフレーム内に置かれている引数を Rtmp にロードし, 
        次に MacroAssembler::store_argument() が生成するコードで
        ロードした値を native の ABI での適切な位置にコピーする.
  
        (なお, ロードする際のフレーム内の位置は Interpreter::local_offset_in_bytes() で計算)」
      ---------------------------------------- -}

	  __ ld(Llocals, Interpreter::local_offset_in_bytes(offset()), Rtmp);
	
	  __ store_argument(Rtmp, jni_arg);
	}
	
```


