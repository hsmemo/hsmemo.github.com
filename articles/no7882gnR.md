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
#define JVM_LEAF(result_type, header)                                \
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (関数定義の関数名および型宣言部が生成される)
      ---------------------------------------- -}

	extern "C" {                                                         \
	  result_type JNICALL header {                                       \

  {- -------------------------------------------
  (1) VM_Exit::block_if_vm_exited() を呼んで
      HotSpot の終了処理が開始されていないかどうかを確認しておく.
      (See: VM_Exit::set_vm_exited())
  
      (なお, もし終了処理が始まっていた場合は, この中で永久にブロックするのでリターンしてこない)
      ---------------------------------------- -}

	    VM_Exit::block_if_vm_exited();                                   \

  {- -------------------------------------------
  (1) __LEAF() マクロのコードが展開される
      ---------------------------------------- -}

	    __LEAF(result_type, header)
	
```


