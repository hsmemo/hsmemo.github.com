---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp

### 名前(function name)
```
bool instanceKlass::find_local_field(Symbol* name, Symbol* sig, fieldDescriptor* fd) const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) fields() の中を線形探索するだけ.
      (name と sig の両方が一致するものがあれば, fd 引数に結果を格納して true を返す. 
      なければ false を返す.)
      ---------------------------------------- -}

	  const int n = fields()->length();
	  for (int i = 0; i < n; i += next_offset ) {
	    int name_index = fields()->ushort_at(i + name_index_offset);
	    int sig_index  = fields()->ushort_at(i + signature_index_offset);
	    Symbol* f_name = constants()->symbol_at(name_index);
	    Symbol* f_sig  = constants()->symbol_at(sig_index);
	    if (f_name == name && f_sig == sig) {
	      fd->initialize(as_klassOop(), i);
	      return true;
	    }
	  }
	  return false;
	}
	
```


