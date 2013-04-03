---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp
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
  (1) VirtualProtect() を呼んで, _polling_page をアクセス不可にする.
    
      (なお, 基本的には失敗しないはずだが, 
       もし失敗したら fatal() で強制終了させている)
      ---------------------------------------- -}

	  DWORD old_status;
	  if( !VirtualProtect((char *)_polling_page, os::vm_page_size(), PAGE_NOACCESS, &old_status) )
	    fatal("Could not disable polling page");
	};
	
```


