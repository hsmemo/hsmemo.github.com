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
address SignatureHandlerLibrary::set_handler(CodeBuffer* buffer) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  address handler   = _handler;
	  int     insts_size = buffer->pure_insts_size();

  {- -------------------------------------------
  (1) もし現状の BufferBlob では大きさが足りないようなら, 
      SignatureHandlerLibrary::set_handler_blob() を呼んで作り直しておく.
      ---------------------------------------- -}

	  if (handler + insts_size > _handler_blob->code_end()) {
	    // get a new handler blob
	    handler = set_handler_blob();
	  }

  {- -------------------------------------------
  (1) もし BufferBlob が存在していれば (NULL でなければ), 
      引数で渡されたコードを memcpy() で BufferBlob 内にコピーする.
      (ついでに, SignatureHandlerLibrary::pd_set_handler() で
       プラットフォーム依存な処理を(もしそんなものがあるなら)行い, 
       ICache::invalidate_range() でコピー範囲の命令キャッシュを invalidate しておく)
    
      その後, コピー先の先頭アドレスをリターン.
      (BufferBlob が存在していなければ NULL をリターン).
      ---------------------------------------- -}

	  if (handler != NULL) {
	    memcpy(handler, buffer->insts_begin(), insts_size);
	    pd_set_handler(handler);
	    ICache::invalidate_range(handler, insts_size);
	    _handler = handler + insts_size;
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return handler;
	}
	
```


