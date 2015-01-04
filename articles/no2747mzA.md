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
void ClassLoader::create_class_path_entry(char *path, struct stat st, ClassPathEntry **new_entry, bool lazy) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  JavaThread* thread = JavaThread::current();

  {- -------------------------------------------
  (1) 条件に応じて, 以下のどれかのオブジェクトを生成し, 
      new_entry 引数で指定された箇所にセットする.
  
      * lazy 引数が true の場合:
        => LazyClassPathEntry オブジェクト
  
      * (lazy 引数が false でかつ) 指定されたパスがファイルである場合(= ディレクトリではない場合)
        => ClassPathZipEntry オブジェクト
  
        (ただし, この場合は JARファイル(= zip ファイル) でないといけないので, 
         もし zip として開けなければ ClassNotFoundException を出す)
        (また, ... の場合には IOException を出す)
  
      * (lazy 引数が false でかつ) 指定されたパスがディレクトリである場合
        => ClassPathDirEntry オブジェクト
      ---------------------------------------- -}

	  if (lazy) {
	    *new_entry = new LazyClassPathEntry(path, st);
	    return;
	  }
	  if ((st.st_mode & S_IFREG) == S_IFREG) {
	    // Regular file, should be a zip file
	    // Canonicalized filename
	    char canonical_path[JVM_MAXPATHLEN];
	    if (!get_canonical_path(path, canonical_path, JVM_MAXPATHLEN)) {
	      // This matches the classic VM
	      EXCEPTION_MARK;
	      THROW_MSG(vmSymbols::java_io_IOException(), "Bad pathname");
	    }
	    char* error_msg = NULL;
	    jzfile* zip;
	    {
	      // enable call to C land
	      ThreadToNativeFromVM ttn(thread);
	      HandleMark hm(thread);
	      zip = (*ZipOpen)(canonical_path, &error_msg);
	    }
	    if (zip != NULL && error_msg == NULL) {
	      *new_entry = new ClassPathZipEntry(zip, path);
	      if (TraceClassLoading) {
	        tty->print_cr("[Opened %s]", path);
	      }
	    } else {
	      ResourceMark rm(thread);
	      char *msg;
	      if (error_msg == NULL) {
	        msg = NEW_RESOURCE_ARRAY(char, strlen(path) + 128); ;
	        jio_snprintf(msg, strlen(path) + 127, "error in opening JAR file %s", path);
	      } else {
	        int len = (int)(strlen(path) + strlen(error_msg) + 128);
	        msg = NEW_RESOURCE_ARRAY(char, len); ;
	        jio_snprintf(msg, len - 1, "error in opening JAR file <%s> %s", error_msg, path);
	      }
	      EXCEPTION_MARK;
	      THROW_MSG(vmSymbols::java_lang_ClassNotFoundException(), msg);
	    }
	  } else {
	    // Directory
	    *new_entry = new ClassPathDirEntry(path);
	    if (TraceClassLoading) {
	      tty->print_cr("[Path %s]", path);
	    }
	  }
	}
	
```


