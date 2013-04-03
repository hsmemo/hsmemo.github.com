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
void TemplateTable::generate_vtable_call(Register Rrecv, Register Rindex, Register Rret) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Register Rtemp = G4_scratch;
	  Register Rcall = Rindex;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert_different_registers(Rcall, G5_method, Gargs, Rret);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // get target methodOop & entry point
	  const int base = instanceKlass::vtable_start_offset() * wordSize;

  {- -------------------------------------------
  (1) コード生成:
      「クラスオブジェクトから vtable を取得し, 
        CPCache から取得しておいた vtable 内の index 番号とつき合わせて methodOop を取得する.
        取得した結果は, G5_method レジスタに格納.」
      ---------------------------------------- -}

	  if (vtableEntry::size() % 3 == 0) {
	    // scale the vtable index by 12:
	    int one_third = vtableEntry::size() / 3;
	    __ sll(Rindex, exact_log2(one_third * 1 * wordSize), Rtemp);
	    __ sll(Rindex, exact_log2(one_third * 2 * wordSize), Rindex);
	    __ add(Rindex, Rtemp, Rindex);
	  } else {
	    // scale the vtable index by 8:
	    __ sll(Rindex, exact_log2(vtableEntry::size() * wordSize), Rindex);
	  }
	
	  __ add(Rrecv, Rindex, Rrecv);
	  __ ld_ptr(Rrecv, base + vtableEntry::method_offset_in_bytes(), G5_method);
	
  {- -------------------------------------------
  (1) コード生成:
      「実際の呼び出し処理を行う」
      ---------------------------------------- -}

	  __ call_from_interpreter(Rcall, Gargs, Rret);
	}
	
```


