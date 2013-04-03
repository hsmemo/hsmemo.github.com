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
#define JNI_ENTRY(result_type, header)                               \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JNI_ENTRY_NO_PRESERVE() マクロのコードが展開される
      ---------------------------------------- -}

	    JNI_ENTRY_NO_PRESERVE(result_type, header)                       \

  {- -------------------------------------------
  (1) (変数宣言など) (See: WeakPreserveExceptionMark)
      ---------------------------------------- -}

	    WeakPreserveExceptionMark __wem(thread);
	
```


