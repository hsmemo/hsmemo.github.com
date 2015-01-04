---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/classLoader.cpp

### 名前(function name)
```
void ClassLoader::load_zip_library() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(ZipOpen == NULL, "should not load zip library twice");

  {- -------------------------------------------
  (1) (libjava がまだロードされていないとまずいので)
      os::native_java_library() を呼んで, libjava が確実にロードされた状態にしておく.
      ---------------------------------------- -}

	  // First make sure native library is loaded
	  os::native_java_library();

  {- -------------------------------------------
  (1) os::dll_load() で libzip をロードする.
      (失敗したら vm_exit_during_initialization() で異常終了)
      ---------------------------------------- -}

	  // Load zip library
	  char path[JVM_MAXPATHLEN];
	  char ebuf[1024];
	  os::dll_build_name(path, sizeof(path), Arguments::get_dll_dir(), "zip");
	  void* handle = os::dll_load(path, ebuf, sizeof ebuf);
	  if (handle == NULL) {
	    vm_exit_during_initialization("Unable to load ZIP library", path);
	  }

  {- -------------------------------------------
  (1) libzip 内の各関数を取得し, 対応する大域変数に格納する (ZipOpen, ZipClose, etc)
      (失敗したら vm_exit_during_initialization() で異常終了.
      なお, ZipClose だけは失敗しても異常終了させていない. 
      これはコメントによると, Windows 上の JDK 5.0 では ZIP_Close が提供されていないため, とのこと)
      ---------------------------------------- -}

	  // Lookup zip entry points
	  ZipOpen      = CAST_TO_FN_PTR(ZipOpen_t, os::dll_lookup(handle, "ZIP_Open"));
	  ZipClose     = CAST_TO_FN_PTR(ZipClose_t, os::dll_lookup(handle, "ZIP_Close"));
	  FindEntry    = CAST_TO_FN_PTR(FindEntry_t, os::dll_lookup(handle, "ZIP_FindEntry"));
	  ReadEntry    = CAST_TO_FN_PTR(ReadEntry_t, os::dll_lookup(handle, "ZIP_ReadEntry"));
	  ReadMappedEntry = CAST_TO_FN_PTR(ReadMappedEntry_t, os::dll_lookup(handle, "ZIP_ReadMappedEntry"));
	  GetNextEntry = CAST_TO_FN_PTR(GetNextEntry_t, os::dll_lookup(handle, "ZIP_GetNextEntry"));
	
	  // ZIP_Close is not exported on Windows in JDK5.0 so don't abort if ZIP_Close is NULL
	  if (ZipOpen == NULL || FindEntry == NULL || ReadEntry == NULL || GetNextEntry == NULL) {
	    vm_exit_during_initialization("Corrupted ZIP library", path);
	  }
	
  {- -------------------------------------------
  (1) libjava 内の Canonicalize() 関数を取得し, 対応する大域変数に格納する.
      ---------------------------------------- -}

	  // Lookup canonicalize entry in libjava.dll
	  void *javalib_handle = os::native_java_library();
	  CanonicalizeEntry = CAST_TO_FN_PTR(canonicalize_fn_t, os::dll_lookup(javalib_handle, "Canonicalize"));
	  // This lookup only works on 1.3. Do not check for non-null here
	}
	
```


