---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateTable_x86_64.cpp

### 名前(function name)
```
void TemplateTable::getstatic(int byte_no) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成: 
      「TemplateTable::getfield_or_static() が生成するコードを実行するだけ」
      ---------------------------------------- -}

	  getfield_or_static(byte_no, true);
	}
	
```


