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
void java_lang_Thread::set_priority(oop java_thread, ThreadPriority priority) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) oopDesc::int_field_put() で priority フィールドに値をセットするだけ.
      ---------------------------------------- -}

	  java_thread->int_field_put(_priority_offset, priority);
	}
	
```


