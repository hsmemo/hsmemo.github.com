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
ReservedHeapSpace::ReservedHeapSpace(const size_t prefix_size,
                                     const size_t prefix_align,
                                     const size_t suffix_size,
                                     const size_t suffix_align,
                                     char* requested_address) :
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ReservedSpace のコンストラクタを呼び出す
      ---------------------------------------- -}

	  ReservedSpace(prefix_size, prefix_align, suffix_size, suffix_align,
	                requested_address,
	                (UseCompressedOops && (Universe::narrow_oop_base() != NULL) &&
	                 Universe::narrow_oop_use_implicit_null_checks()) ?
	                  lcm(os::vm_page_size(), prefix_align) : 0) {

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

	  protect_noaccess_prefix(prefix_size+suffix_size);
	}
	
```


