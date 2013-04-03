---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.hpp


### 本体部(body)
```
  {- -------------------------------------------
  (1) 「ネイティブコードの先頭アドレスを格納しているスロット」のアドレスをリターン.
  
      (ネイティブコードの先頭アドレス自体はクラス宣言中でフィールドとして宣言されていない 
       (というか全てのフィールドが終わった後に格納されている) ので, 
       "(this+1)" のアドレスをリターン.)
      (See: methodOopDesc)
      ---------------------------------------- -}

	  address* native_function_addr() const          { assert(is_native(), "must be native"); return (address*) (this+1); }
	
```


