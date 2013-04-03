---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp

### 名前(function name)
```
static bool linux_mprotect(char* addr, size_t size, int prot) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) ::mprotect() を呼び出すだけ.
      (ただし, アドレスは page size で align しておかないといけないため, 補正する処理も行っている.
  
       といっても開始アドレスについては, 
       SUSv3 の規定で mprotect() が使えるのは mmap() で取得したメモリ領域だけとされており, 
       mmap() で取得したメモリなら align されているはずで, 
       だから align されてなかったら HotSpot 側のバグなので見直せ, 
       とコメントには書かれている)
      ---------------------------------------- -}

	  // Linux wants the mprotect address argument to be page aligned.
	  char* bottom = (char*)align_size_down((intptr_t)addr, os::Linux::page_size());
	
	  // According to SUSv3, mprotect() should only be used with mappings
	  // established by mmap(), and mmap() always maps whole pages. Unaligned
	  // 'addr' likely indicates problem in the VM (e.g. trying to change
	  // protection of malloc'ed or statically allocated memory). Check the
	  // caller if you hit this assert.
	  assert(addr == bottom, "sanity check");
	
	  size = align_size_up(pointer_delta(addr, bottom, 1) + size, os::Linux::page_size());
	  return ::mprotect(bottom, size, prot) == 0;
	}
	
```


