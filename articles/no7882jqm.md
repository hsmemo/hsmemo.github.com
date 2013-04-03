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
void TemplateTable::putfield(int byte_no) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コード生成: 
      「TemplateTable::putfield_or_static() が生成するコードを実行するだけ」
      ---------------------------------------- -}

	  putfield_or_static(byte_no, false);
	}
	
```


