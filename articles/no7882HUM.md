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
#define JRT_BLOCK                                                    \
    {                                                                \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      (See: ThreadInVMfromJava)
      ---------------------------------------- -}

	    ThreadInVMfromJava __tiv(thread);                                \
	    Thread* THREAD = thread;                                         \

  {- -------------------------------------------
  (1) (変数宣言など) (デバッグ用の処理)
      (See: VMEntryWrapper)
      ---------------------------------------- -}

	    debug_only(VMEntryWrapper __vew;)
	
```


