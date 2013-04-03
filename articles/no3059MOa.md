---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.cpp

### 名前(function name)
```
address TemplateInterpreterGenerator::generate_error_exit(const char* msg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (ここからが生成するコードの始まり)
      ---------------------------------------- -}

	  address entry = __ pc();

  {- -------------------------------------------
  (1) コード生成:
      「強制的に停止させる」
      ---------------------------------------- -}

	  __ stop(msg);

  {- -------------------------------------------
  (1) 以上で生成したコードの先頭アドレスをリターン.
      ---------------------------------------- -}

	  return entry;
	}
	
```


