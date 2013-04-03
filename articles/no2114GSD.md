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
void* Management::get_jmm_interface(int version) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) jmm_interface (static に確保している jmm 関数ポインタの詰まった構造体) へのポインタを返す.
  
      (ただし, 引数で渡されたバージョンが JMM_VERSION_1_0 でなければ NULL を返す)
      ---------------------------------------- -}

	  if (version == JMM_VERSION_1_0) {
	    return (void*) &jmm_interface;
	  }
	  return NULL;
	}
	
```


