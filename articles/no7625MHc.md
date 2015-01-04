---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/sun/launcher/LauncherHelper.java

### 名前(function name)
```
    static String getMainClassFromJar(PrintStream ostream, String jarname) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まず, jarname 引数に対応する JarFile オブジェクトを生成する.
      その後, getManifest() で Manifest オブジェクトを取得し, 
      そこから getMainAttributes() で Attributes オブジェクトを取得する.
      最後に, getValue() でメインクラス名を取得し, それをリターン.
  
      (なお, Manifest オブジェクトや Attributes オブジェクトが取得できなかった場合は abort().
       また, IOException が出た場合も abort())
      ---------------------------------------- -}

	        try {
	            JarFile jarFile = null;
	            try {
	                jarFile = new JarFile(jarname);
	                Manifest manifest = jarFile.getManifest();
	                if (manifest == null) {
	                    abort(ostream, null, "java.launcher.jar.error2", jarname);
	                }
	                Attributes mainAttrs = manifest.getMainAttributes();
	                if (mainAttrs == null) {
	                    abort(ostream, null, "java.launcher.jar.error3", jarname);
	                }
	                return mainAttrs.getValue(MAIN_CLASS).trim();
	            } finally {
	                if (jarFile != null) {
	                    jarFile.close();
	                }
	            }
	        } catch (IOException ioe) {
	            abort(ostream, ioe, "java.launcher.jar.error1", jarname);
	        }
	        return null;
	    }
	
```


