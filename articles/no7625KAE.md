---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvm.cpp

### 名前(function name)
```
JVM_ENTRY(jclass, JVM_DefineClassWithSource(JNIEnv *env, const char *name, jobject loader, const jbyte *buf, jsize len, jobject pd, const char *source))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力) (See: JVMWrapper2)
      ---------------------------------------- -}

	  JVMWrapper2("JVM_DefineClassWithSource %s", name);
	
  {- -------------------------------------------
  (1) jvm_define_class_common() を呼んでクラスのロード処理を行い, 結果をリターン.
      ---------------------------------------- -}

	  return jvm_define_class_common(env, name, loader, buf, len, pd, source, true, THREAD);
	JVM_END
	
```


