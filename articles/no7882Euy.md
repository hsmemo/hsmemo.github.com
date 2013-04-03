---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp
### 説明(description)

```
// Mark the polling page as unreadable
```

### 名前(function name)
```
void os::make_polling_page_unreadable(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) os::guard_memory() を呼んで, _polling_page をアクセス不可にする.
    
      (なお, 基本的には失敗しないはずだが, 
       もし失敗したら fatal() で強制終了させている)
      ---------------------------------------- -}

	  if( !guard_memory((char*)_polling_page, Linux::page_size()) )
	    fatal("Could not disable polling page");
	};
	
```


