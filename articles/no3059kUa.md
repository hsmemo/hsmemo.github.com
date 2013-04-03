---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)

```
// Stub to set the value of the real pointer, and then call the real
// function.

```

### 名前(function name)
```
static long priocntl_stub(int pcver, idtype_t idtype, id_t id, int cmd, caddr_t arg) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) dylsym() で "__priocntl"(priocntl) をロードして, priocntl_ptr 変数にセットする.
      その後, ロードした __priocntl() を呼び出す.
      ---------------------------------------- -}

	  // Try Solaris 8- name only.
	  priocntl_type tmp = (priocntl_type)dlsym(RTLD_DEFAULT, "__priocntl");
	  guarantee(tmp != NULL, "priocntl function not found.");
	  priocntl_ptr = tmp;
	  return (*priocntl_ptr)(PC_VERSION, idtype, id, cmd, arg);
	}
	
```


