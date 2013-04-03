---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/codeCache.cpp

### 名前(function name)
```
  void print(const char* title) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 集めた情報を出力する
      ---------------------------------------- -}

	    tty->print_cr(" #%d %s = %dK (hdr %d%%,  loc %d%%, code %d%%, stub %d%%, [oops %d%%, data %d%%, pcs %d%%])",
	                  count,
	                  title,
	                  total() / K,
	                  header_size             * 100 / total_size,
	                  relocation_size         * 100 / total_size,
	                  code_size               * 100 / total_size,
	                  stub_size               * 100 / total_size,
	                  scopes_oop_size         * 100 / total_size,
	                  scopes_data_size        * 100 / total_size,
	                  scopes_pcs_size         * 100 / total_size);
	  }
	
```


