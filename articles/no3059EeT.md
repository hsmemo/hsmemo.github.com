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
void SignatureHandlerLibrary::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 既に初期化済みであれば, 何もする必要が無いのでここでリターン.
      ---------------------------------------- -}

	  if (_fingerprints != NULL) {
	    return;
	  }

  {- -------------------------------------------
  (1) SignatureHandlerLibrary 内のフィールドの初期化処理.
      ---------------------------------------- -}

	  if (set_handler_blob() == NULL) {
	    vm_exit_out_of_memory(blob_size, "native signature handlers");
	  }
	
	  BufferBlob* bb = BufferBlob::create("Signature Handler Temp Buffer",
	                                      SignatureHandlerLibrary::buffer_size);
	  _buffer = bb->code_begin();
	
	  _fingerprints = new(ResourceObj::C_HEAP)GrowableArray<uint64_t>(32, true);
	  _handlers     = new(ResourceObj::C_HEAP)GrowableArray<address>(32, true);
	}
	
```


