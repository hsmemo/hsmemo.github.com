---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/javaClasses.cpp

### 名前(function name)
```
void java_lang_Thread::set_thread(oop java_thread, JavaThread* thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopDesc::address_field_put() で eetop フィールドに値をセットするだけ.
      (java_lang_Thread::_eetop_offset を引数として呼び出す)
      ---------------------------------------- -}

	  java_thread->address_field_put(_eetop_offset, (address)thread);
	}
	
```


