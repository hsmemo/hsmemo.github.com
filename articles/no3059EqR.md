---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/exceptions.cpp

### 名前(function name)
```
void Exceptions::_throw_msg(Thread* thread, const char* file, int line, Symbol* h_name, const char* message, Handle h_loader, Handle h_protection_domain) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) Exceptions::special_exception() を呼んで, 特殊ケースかどうかをチェックしておく.
      もし処理しなくていいケースなら, ここでリターン.
      ---------------------------------------- -}

	  // Check for special boot-strapping/vm-thread handling
	  if (special_exception(thread, file, line, h_name, message)) return;

  {- -------------------------------------------
  (1) Exceptions::new_exception() で新しい例外オブジェクトを作った後, 
      Exceptions::_throw() で pending_exception フィールドにセットする.
      ---------------------------------------- -}

	  // Create and throw exception
	  Handle h_cause(thread, NULL);
	  Handle h_exception = new_exception(thread, h_name, message, h_cause, h_loader, h_protection_domain);
	  _throw(thread, file, line, h_exception, message);
	}
	
```


