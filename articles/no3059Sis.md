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
void SignatureIterator::expect(char c) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 予想と外れていたら fatal(). 
      逆に予想が合っていたら, _index をその分だけ (= 1文字だけ) 進めておく.
      ---------------------------------------- -}

	  if (_signature->byte_at(_index) != c) fatal(err_msg("expecting %c", c));
	  _index++;
	}
	
```


