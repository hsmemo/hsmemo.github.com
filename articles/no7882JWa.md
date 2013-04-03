---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp
### 説明(description)

```
// The Rcache register must be set before call
```

### 名前(function name)
```
void TemplateTable::load_field_cp_cache_entry(Register Robj,
                                              Register Rcache,
                                              Register index,
                                              Register Roffset,
                                              Register Rflags,
                                              bool is_static) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rcache, Rflags, Roffset);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ByteSize cp_base_offset = constantPoolCacheOopDesc::base_offset();
	
  {- -------------------------------------------
  (1) コード生成:
      「CPcache エントリ中から, 以下の情報をそれぞれ取得して, レジスタに格納する.
        * _flags 情報 -> Rflags レジスタ
        * _f2 情報 -> Roffset レジスタ
        * _f1 情報 -> Robj レジスタ (ただしこれは is_static が true の場合のみ取得)」
      ---------------------------------------- -}

	  __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::flags_offset(), Rflags);
	  __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::f2_offset(), Roffset);
	  if (is_static) {
	    __ ld_ptr(Rcache, cp_base_offset + ConstantPoolCacheEntry::f1_offset(), Robj);
	  }
	}
	
```


