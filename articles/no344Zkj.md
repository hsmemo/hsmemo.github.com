---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/virtualspace.cpp

### 名前(function name)
```
ReservedHeapSpace::ReservedHeapSpace(size_t size, size_t alignment,
                                     bool large, char* requested_address) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) スーパークラスの初期化
      ---------------------------------------- -}

	  ReservedSpace(size, alignment, large,
	                requested_address,
	                (UseCompressedOops && (Universe::narrow_oop_base() != NULL) &&
	                 Universe::narrow_oop_use_implicit_null_checks()) ?
	                  lcm(os::vm_page_size(), alignment) : 0) {

  {- -------------------------------------------
  (1) protect_noaccess_prefix() を呼び出す.
        (protect_noaccess_prefix() 内の処理により, 
         もし Compressed Oop オプションが有効で (UseCompressedOops), 
         かつヒープの開始アドレスが 0 でなく (= Universe::narrow_oop_base() != NULL) , 
         さらに null check を暗黙的に行うことになっていれば (= Universe::narrow_oop_use_implicit_null_checks() が true), 
         確保したヒープの先頭ページがアクセス禁止にされる.
         これにより Null Pointer アクセスがシグナルハンドラで検出できるようになる.
         See: protect_noaccess_prefix())
      ---------------------------------------- -}

	  // Only reserved space for the java heap should have a noaccess_prefix
	  // if using compressed oops.
	  protect_noaccess_prefix(size);
	}
	
```


