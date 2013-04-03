---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/signature.cpp

### 名前(function name)
```
void SignatureIterator::check_signature_end() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) signature string の終端に達していることを確認する.
      もし達していなかったら, エラーメッセージを出力.
      ---------------------------------------- -}

	  if (_index < _signature->utf8_length()) {
	    tty->print_cr("too many chars in signature");
	    _signature->print_value_on(tty);
	    tty->print_cr(" @ %d", _index);
	  }
	}
	
```


