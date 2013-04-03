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
// Mark the polling page as readable
```

### 名前(function name)
```
void os::make_polling_page_readable(void) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) linux_mprotect() を呼んで, _polling_page をアクセス可能な状態にする.
    
      (なお, 基本的には失敗しないはずだが, 
       もし失敗したら fatal() で強制終了させている)
      ---------------------------------------- -}

	  if( !linux_mprotect((char *)_polling_page, Linux::page_size(), PROT_READ)) {
	    fatal("Could not enable polling page");
	  }
	};
	
```


