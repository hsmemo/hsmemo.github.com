---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/net/URLClassLoader.java
### 説明(description)

```
    /*
     * Defines a Class using the class bytes obtained from the specified
     * Resource. The resulting Class must be resolved before it can be
     * used.
     */
```

### 名前(function name)
```
    private Class defineClass(String name, Resource res) throws IOException {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	        long t0 = System.nanoTime();
	        int i = name.lastIndexOf('.');
	        URL url = res.getCodeSourceURL();

  {- -------------------------------------------
  (1) name 引数で指定されたクラス名にパッケージ部分が含まれる場合, 
      getAndVerifyPackage() でチェックしておく.
  
      また, 初めて見るパッケージ名の場合は
      java.net.URLClassLoader.definePackage() または java.lang.ClassLoader.definePackage() で登録しておく.
      ---------------------------------------- -}

	        if (i != -1) {
	            String pkgname = name.substring(0, i);
	            // Check if package already loaded.
	            Manifest man = res.getManifest();
	            if (getAndVerifyPackage(pkgname, man, url) == null) {
	                try {
	                    if (man != null) {
	                        definePackage(pkgname, man, url);
	                    } else {
	                        definePackage(pkgname, null, null, null, null, null, null, null);
	                    }
	                } catch (IllegalArgumentException iae) {
	                    // parallel-capable class loaders: re-verify in case of a
	                    // race condition
	                    if (getAndVerifyPackage(pkgname, man, url) == null) {
	                        // Should never happen
	                        throw new AssertionError("Cannot find package " +
	                                                 pkgname);
	                    }
	                }
	            }
	        }

  {- -------------------------------------------
  (1) java.lang.ClassLoader.defineClass() を呼んで, ロードを行う.
  
      なお, ClassLoader.defineClass() には二種類ある
      (byte[] を使うものと java.nio.ByteBuffer を使うもの. なお, java.nio.ByteBuffer を使う方は 1.5 以降の API).
      指定のクラスファイル(Resource オブジェクト)から ByteBuffer が取得できなければ前者を, 取得できれば後者を呼び出している.
      (ただし, どちらもそんなに大きな違いはなく, 最終的には native method である defineClass1() または defineClass2() 経由で
       CVMI 関数である JVM_DefineClassWithSource() を呼び出す)
      ---------------------------------------- -}

	        // Now read the class bytes and define the class
	        java.nio.ByteBuffer bb = res.getByteBuffer();
	        if (bb != null) {
	            // Use (direct) ByteBuffer:
	            CodeSigner[] signers = res.getCodeSigners();
	            CodeSource cs = new CodeSource(url, signers);
	            sun.misc.PerfCounter.getReadClassBytesTime().addElapsedTimeFrom(t0);
	            return defineClass(name, bb, cs);
	        } else {
	            byte[] b = res.getBytes();
	            // must read certificates AFTER reading bytes.
	            CodeSigner[] signers = res.getCodeSigners();
	            CodeSource cs = new CodeSource(url, signers);
	            sun.misc.PerfCounter.getReadClassBytesTime().addElapsedTimeFrom(t0);
	            return defineClass(name, b, 0, b.length, cs);
	        }
	    }
	
```


