---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Dump heap - Returns 0 if succeeds.
```

### 名前(function name)
```
JVM_ENTRY(jint, jmm_DumpHeap0(JNIEnv *env, jstring outputfile, jboolean live))
```

### 本体部(body)
```
	#ifndef SERVICES_KERNEL

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(THREAD);

  {- -------------------------------------------
  (1) 引数が NULL だった場合は, NullPointerException.
      ---------------------------------------- -}

	  oop on = JNIHandles::resolve_external_guard(outputfile);
	  if (on == NULL) {
	    THROW_MSG_(vmSymbols::java_lang_NullPointerException(),
	               "Output file name cannot be null.", -1);
	  }
	  char* name = java_lang_String::as_utf8_string(on);
	  if (name == NULL) {
	    THROW_MSG_(vmSymbols::java_lang_NullPointerException(),
	               "Output file name cannot be null.", -1);
	  }

  {- -------------------------------------------
  (1) HeapDumper::dumper() を呼び出して, ダンプを出力する.
      もし失敗したら IOException.
      成功したら 0 でリターン.
      ---------------------------------------- -}

	  HeapDumper dumper(live ? true : false);
	  if (dumper.dump(name) != 0) {
	    const char* errmsg = dumper.error_as_C_string();
	    THROW_MSG_(vmSymbols::java_io_IOException(), errmsg, -1);
	  }
	  return 0;

  {- -------------------------------------------
  (1) (なお, #define SERVICES_KERNEL 時には, -1 を返すだけ)
      ---------------------------------------- -}

	#else  // SERVICES_KERNEL
	  return -1;
	#endif // SERVICES_KERNEL
	JVM_END
	
```


