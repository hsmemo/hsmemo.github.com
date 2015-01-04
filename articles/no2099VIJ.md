---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/bin/java.c
### 説明(description)

```
/*
 * Loads a class and verifies that the main class is present and it is ok to
 * call it for more details refer to the java implementation.
 */
```

### 名前(function name)
```
static jclass
LoadMainClass(JNIEnv *env, int mode, char *name)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    jmethodID mid;
	    jstring str;
	    jobject result;
	    jlong start, end;

  {- -------------------------------------------
  (1) ブートストラップクラスローダーを使って "sun.launcher.LauncherHelper" クラスを取得する.
      (もし結果が NULL だったら, ここでリターン)
      ---------------------------------------- -}

	    jclass cls = GetLauncherHelperClass(env);
	    NULL_CHECK0(cls);

  {- -------------------------------------------
  (1) (トレース出力用の処理)
      ---------------------------------------- -}

	    if (JLI_IsTraceLauncher()) {
	        start = CounterGet();
	    }

  {- -------------------------------------------
  (1) 取得した sun.launcher.LauncherHelper クラス内から, checkAndLoadMain() メソッドを取得する.
      (もし結果が NULL だったら, ここでリターン)
      ---------------------------------------- -}

	    NULL_CHECK0(mid = (*env)->GetStaticMethodID(env, cls,
	                "checkAndLoadMain",
	                "(ZILjava/lang/String;)Ljava/lang/Class;"));
	
  {- -------------------------------------------
  (1) 取得した sun.launcher.LauncherHelper.checkAndLoadMain() メソッドを呼び出す.
      ---------------------------------------- -}

	    switch (mode) {
	        case LM_CLASS:
	            str = NewPlatformString(env, name);
	            break;
	        default:
	            str = (*env)->NewStringUTF(env, name);
	            break;
	    }
	    result = (*env)->CallStaticObjectMethod(env, cls, mid, USE_STDERR, mode, str);
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    if (JLI_IsTraceLauncher()) {
	        end   = CounterGet();
	        printf("%ld micro seconds to load main class\n",
	               (long)(jint)Counter2Micros(end-start));
	        printf("----_JAVA_LAUNCHER_DEBUG----\n");
	    }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	    return (jclass)result;
	}
	
```


