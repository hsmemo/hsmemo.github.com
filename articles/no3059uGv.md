---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os_cpu/linux_x86/vm/os_linux_x86.cpp

### 名前(function name)
```
void os::Linux::set_fpu_control_word(int fpu_control) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) x86_64 の場合には, 何もしない.
  
      x86_32 の場合には, _FPU_SETCW() を呼び出して 
      x87 の control word を引数で指定された値に変更する.
      ---------------------------------------- -}

	#ifndef AMD64
	  _FPU_SETCW(fpu_control);
	#endif // !AMD64
	}
	
```


