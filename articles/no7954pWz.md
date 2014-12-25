---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/native/java/lang/Class.c

### 名前(function name)
```
JNIEXPORT jclass JNICALL
Java_java_lang_Class_forName0(JNIEnv *env, jclass this, jstring classname,
                              jboolean initialize, jobject loader)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    char *clname;
	    jclass cls = 0;
	    char buf[128];
	    jsize len;
	    jsize unicode_len;
	
  {- -------------------------------------------
  (1) classname 引数が不正な場合 (NULL の場合) は NullPointerException.
      ---------------------------------------- -}

	    if (classname == NULL) {
	        JNU_ThrowNullPointerException(env, 0);
	        return 0;
	    }
	
  {- -------------------------------------------
  (1) 作業用に使用するバッファを設定し(clname 局所変数), classname 引数の内容をコピーしておく.
      
      予め用意したバッファ(buf局所変数)で足りる場合は, それを用いる.
      長さが足りない場合は, malloc() を呼んで新しくバッファを確保する.
      (なお, malloc() が失敗した場合は OutOfMemoryError)
      ---------------------------------------- -}

	    len = (*env)->GetStringUTFLength(env, classname);
	    unicode_len = (*env)->GetStringLength(env, classname);
	    if (len >= (jsize)sizeof(buf)) {
	        clname = malloc(len + 1);
	        if (clname == NULL) {
	            JNU_ThrowOutOfMemoryError(env, NULL);
	            return NULL;
	        }
	    } else {
	        clname = buf;
	    }
	    (*env)->GetStringUTFRegion(env, classname, 0, unicode_len, clname);
	
  {- -------------------------------------------
  (1) VerifyFixClassname() を呼んで, クラス名中の '.' を '/' に置換する.
      なお, classname 引数が不正な場合 (初めから '/' が混じっている場合) は ClassNotFoundException.
      ---------------------------------------- -}

	    if (VerifyFixClassname(clname) == JNI_TRUE) {
	        /* slashes present in clname, use name b4 translation for exception */
	        (*env)->GetStringUTFRegion(env, classname, 0, unicode_len, clname);
	        JNU_ThrowClassNotFoundException(env, clname);
	        goto done;
	    }
	
  {- -------------------------------------------
  (1) classname 引数が不正な場合 (妥当なクラス名ではない文字列の場合) は ClassNotFoundException.
      ---------------------------------------- -}

	    if (!VerifyClassname(clname, JNI_TRUE)) {  /* expects slashed name */
	        JNU_ThrowClassNotFoundException(env, clname);
	        goto done;
	    }
	
  {- -------------------------------------------
  (1) JVM_FindClassFromClassLoader() を呼んで, 引数で指定されたクラスオブジェクトを取得する.
      ---------------------------------------- -}

	    cls = JVM_FindClassFromClassLoader(env, clname, initialize,
	                                       loader, JNI_FALSE);
	
  {- -------------------------------------------
  (1) (必要があれば) malloc() で確保したバッファを解放しておく
      ---------------------------------------- -}

	 done:
	    if (clname != buf) {
	        free(clname);
	    }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	    return cls;
	}
	
```


