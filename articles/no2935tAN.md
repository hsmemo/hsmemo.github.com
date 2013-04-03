---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp

### 名前(function name)
```
void JvmtiManageCapabilities::copy_capabilities(const jvmtiCapabilities *from, jvmtiCapabilities *to) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) from 引数が指す箇所から to 引数が指す箇所にメモリコピーをするだけ.
      ---------------------------------------- -}

	  char *ap = (char *)from;
	  char *resultp = (char *)to;
	
	  for (int i = 0; i < CAPA_SIZE; ++i) {
	    *resultp++ = *ap++;
	  }
	}
	
```


