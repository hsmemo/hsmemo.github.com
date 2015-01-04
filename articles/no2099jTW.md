---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/solaris/bin/java_md.c

### 名前(function name)
```
jclass
FindBootStrapClass(JNIEnv *env, const char* classname)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JVM_FindClassFromBootLoader() を呼んで, classname 引数で指定されたクラスをロードする.
      
      (なお, JVM_FindClassFromBootLoader() は dlsym() で実行時に取得する.
       一度取得したら findBootClass 変数に束縛し, 以降はそれを流用している.
       また, dlsym() が失敗した場合は, JLI_ReportErrorMessage() でトレース出力を出した後, NULL をリターン).
      ---------------------------------------- -}

	   if (findBootClass == NULL) {
	       findBootClass = (FindClassFromBootLoader_t *)dlsym(RTLD_DEFAULT,
	          "JVM_FindClassFromBootLoader");
	       if (findBootClass == NULL) {
	           JLI_ReportErrorMessage(DLL_ERROR4,
	               "JVM_FindClassFromBootLoader");
	           return NULL;
	       }
	   }
	   return findBootClass(env, classname);
	}
	
```


