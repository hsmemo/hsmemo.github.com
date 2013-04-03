---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp
### 説明(description)
// Get the method data pointer from the methodOop and set the
// specified register to its value.



### 名前(function name)
```
void InterpreterMacroAssembler::set_method_data_pointer() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ProfileInterpreter, "must be profiling interpreter");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Label get_continue;
	
  {- -------------------------------------------
  (1) コード生成:
      「Lmethod レジスタに入っている methodOop に 
       methodDataOop オブジェクトが格納されているかどうかを確認する.
  
       もし methodDataOop があれば, 
       そこから DataLayout を取りだして ImethodDataPtr にセットする.
       (逆に methodDataOop が無ければ, ImethodDataPtr の値は NULL になる)」
      ---------------------------------------- -}

	  ld_ptr(Lmethod, in_bytes(methodOopDesc::method_data_offset()), ImethodDataPtr);
	  test_method_data_pointer(get_continue);
	  add(ImethodDataPtr, in_bytes(methodDataOopDesc::data_offset()), ImethodDataPtr);
	  bind(get_continue);
	}
	
```


