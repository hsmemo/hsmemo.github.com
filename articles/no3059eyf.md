---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp

### 名前(function name)
```
address SignatureHandlerLibrary::set_handler_blob() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) BufferBlob::create() で BufferBlob を生成する.
      もし失敗したら NULL をリターン.
      逆に成功したら, _handler_blob フィールドや _handler フィールドに生成した BufferBlob をセットした後, 
      その BufferBlob の先頭アドレスをリターンする.
      ---------------------------------------- -}

	  BufferBlob* handler_blob = BufferBlob::create("native signature handlers", blob_size);
	  if (handler_blob == NULL) {
	    return NULL;
	  }
	  address handler = handler_blob->code_begin();
	  _handler_blob = handler_blob;
	  _handler = handler;
	  return handler;
	}
	
```


