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
JavaThread* java_lang_Thread::thread(oop java_thread) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopDesc::address_field() で eetop フィールドの値を取得し, それをリターンするだけ.
      (java_lang_Thread::_eetop_offset を引数として呼び出す)
      ---------------------------------------- -}

	  return (JavaThread*)java_thread->address_field(_eetop_offset);
	}
	
```


