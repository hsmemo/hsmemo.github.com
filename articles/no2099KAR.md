---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/windows/bin/java_md.c

### 名前(function name)
```
jclass FindBootStrapClass(JNIEnv *env, const char *classname)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	   HMODULE hJvm;
	
  {- -------------------------------------------
  (1) JVM_FindClassFromBootLoader() を呼んで, classname 引数で指定されたクラスをロードする.
      
      (なお, JVM_FindClassFromBootLoader() は GetProcAddress() で実行時に取得する.
       一度取得したら findBootClass 変数に束縛し, 以降はそれを流用している.
       また, GetProcAddress() が失敗した場合は, JLI_ReportErrorMessage() でトレース出力を出した後, NULL をリターン).
      ---------------------------------------- -}

	   if (findBootClass == NULL) {
	       hJvm = GetModuleHandle(JVM_DLL);
	       if (hJvm == NULL) return NULL;
	       /* need to use the demangled entry point */
	       findBootClass = (FindClassFromBootLoader_t *)GetProcAddress(hJvm,
	            "JVM_FindClassFromBootLoader");
	       if (findBootClass == NULL) {
	          JLI_ReportErrorMessage(DLL_ERROR4, "JVM_FindClassFromBootLoader");
	          return NULL;
	       }
	   }
	   return findBootClass(env, classname);
	}
	
```


