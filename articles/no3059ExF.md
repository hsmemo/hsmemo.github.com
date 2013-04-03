---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::load_invoke_cp_cache_entry(int byte_no,
                                               Register Rmethod,
                                               Register Ritable_index,
                                               Register Rflags,
                                               bool is_invokevirtual,
                                               bool is_invokevfinal,
                                               bool is_invokedynamic) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Uses both G3_scratch and G4_scratch
	  Register Rcache = G3_scratch;
	  Register Rscratch = G4_scratch;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rcache, Rmethod, Ritable_index);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ByteSize cp_base_offset = constantPoolCacheOopDesc::base_offset();
	
	  // determine constant pool cache field offsets
	  const int method_offset = in_bytes(
	    cp_base_offset +
	      (is_invokevirtual
	       ? ConstantPoolCacheEntry::f2_offset()
	       : ConstantPoolCacheEntry::f1_offset()
	      )
	    );
	  const int flags_offset = in_bytes(cp_base_offset +
	                                    ConstantPoolCacheEntry::flags_offset());
	  // access constant pool cache fields
	  const int index_offset = in_bytes(cp_base_offset +
	                                    ConstantPoolCacheEntry::f2_offset());
	
  {- -------------------------------------------
  (1) コード生成:
      「対応する CPCache entry をロードした後 (ロード先は Rcache), 
        引数の Rmethod で指定されたレジスタに, CPCache entry の f1 または f2 フィールドの値をロード(??? #TODO)」
       (invokevirtual の場合には f2 フィールド, そうでない場合には f1 フィールドをロード)
    
       (なお, invokevfinal の場合には, ...#TODO)
       (また, 引数の byte_no が...の場合には, ...#TODO)
      ---------------------------------------- -}

	  if (is_invokevfinal) {
	    __ get_cache_and_index_at_bcp(Rcache, Rscratch, 1);
	    __ ld_ptr(Rcache, method_offset, Rmethod);
	  } else if (byte_no == f1_oop) {
	    // Resolved f1_oop goes directly into 'method' register.
	    resolve_cache_and_index(byte_no, Rmethod, Rcache, Rscratch, sizeof(u4));
	  } else {
	    resolve_cache_and_index(byte_no, noreg, Rcache, Rscratch, sizeof(u2));
	    __ ld_ptr(Rcache, method_offset, Rmethod);
	  }
	
  {- -------------------------------------------
  (1) コード生成: (ただし, 引数の Ritable_index が noreg の場合には生成しない)
      「引数の Ritable_index で指定されたレジスタに, CPCache entry の f2 フィールドの値をロード」
      ---------------------------------------- -}

	  if (Ritable_index != noreg) {
	    __ ld_ptr(Rcache, index_offset, Ritable_index);
	  }

  {- -------------------------------------------
  (1) コード生成:
      「引数の Rflags で指定されたレジスタに, CPCache entry の flags フィールドの値をロード」
      ---------------------------------------- -}

	  __ ld_ptr(Rcache, flags_offset, Rflags);
	}
	
```


