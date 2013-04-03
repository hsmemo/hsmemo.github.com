---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/interfaceSupport.hpp

### 名前(function name)
```
#define JRT_ENTRY(result_type, header)                               \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (関数定義の関数名および型宣言部が生成される)
      ---------------------------------------- -}

	  result_type header {                                               \

  {- -------------------------------------------
  (1) (変数宣言など)
      (See: ThreadInVMfromJava)
      ---------------------------------------- -}

	    ThreadInVMfromJava __tiv(thread);                                \

  {- -------------------------------------------
  (1) __ENTRY() マクロのコードが展開される
      ---------------------------------------- -}

	    __ENTRY(result_type, header, thread)                             \

  {- -------------------------------------------
  (1) (変数宣言など) (デバッグ用の処理)
      (See: VMEntryWrapper)
      ---------------------------------------- -}

	    debug_only(VMEntryWrapper __vew;)
	
```


