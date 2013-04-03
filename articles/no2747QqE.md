---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp

### 名前(function name)
```
  static void print() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) SystemDictionary::classes_do() と SystemDictionary::methods_do() でクラス情報を取得し, 結果を出力する.
      ---------------------------------------- -}

	    SystemDictionary::classes_do(do_class);
	    SystemDictionary::methods_do(do_method);
	    tty->print_cr("Class statistics:");
	    tty->print_cr("%d classes (%d bytes)", nclasses, class_size * oopSize);
	    tty->print_cr("%d methods (%d bytes = %d base + %d debug info)", nmethods,
	                  (method_size + debug_size) * oopSize, method_size * oopSize, debug_size * oopSize);
	    tty->print_cr("%d methoddata (%d bytes)", nmethoddata, methoddata_size * oopSize);
	
```


