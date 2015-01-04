---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理の開始点 ： メインクラスのロード処理
---
[Up](no7ggAHQj6.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理の開始点 ： メインクラスのロード処理

--- 
## 概要(Summary)
java コマンド (launcher) は, JavaVM の初期化が完了した後,
LoadMainClass() を呼び出すことでメインクラスのロードを行う (See: [here](nokAq80V3Y.html) for details).

LoadMainClass() でのロード処理は, ほとんどが
sun.launcher.LauncherHelper.checkAndLoadMain() メソッドで行われている.
sun.launcher.LauncherHelper.checkAndLoadMain() メソッドの処理は大きくは以下の 3つからなり, 
このうち 2. の箇所で JNI 経由で Java コードレベルでのシステムクラスローダーによるロード処理が呼び出され,
最終的には ClassLoader.defineClass() でロード処理が開始される.

1. 必要があれば, Jar ファイル中の manifest からクラス名を取得する.

2. System ClassLoader を用いてクラスをロードする.

3. main() メソッドの存在をチェックする.


```java
    ((cite: jdk/src/share/classes/sun/launcher/LauncherHelper.java))
         * This method does the following:
         * 1. gets the classname from a Jar's manifest, if necessary
         * 2. loads the class using the System ClassLoader
         * 3. ensures the availability and accessibility of the main method,
         *    using signatureDiagnostic method.
         *    a. does the class exist
         *    b. is there a main
         *    c. is the main public
         *    d. is the main static
         *    c. does the main take a String array for args
         * 4. and off we go......
```

## 備考(Notes)
仕様上の言及箇所は, Java仮想マシン仕様 "5.2 Virtual Machine Start-up" 参照.


## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: [here](nokAq80V3Y.html) for details)
-> LoadMainClass()
   -> (1) ブートストラップクラスローダーを使って "sun.launcher.LauncherHelper" クラスを取得する.
          -> GetLauncherHelperClass()
             -> FindBootStrapClass()
                -> dlsym() or GetProcAddress()  (<= JVM_FindClassFromBootLoader() を取得)
                -> JVM_FindClassFromBootLoader()
                   -> SystemDictionary::resolve_or_null()
                      -> (See: [here](noIvSV0NZj.html) for details)
      (2) sun.launcher.LauncherHelper クラスの checkAndLoadMain() メソッドを取得する.
          -> jni_GetStaticMethodID()
             -> (See: [here](no3059-0k.html) for details)
      (3) sun.launcher.LauncherHelper クラスの checkAndLoadMain() メソッドを呼んで, メインクラスのロードを行う.
          -> jni_CallStaticObjectMethod()
             -> (See: [here](no3059-0k.html) for details)
                -> sun.launcher.LauncherHelper.checkAndLoadMain()
                   -> (1) システムクラスローダを取得する.
                          -> java.lang.ClassLoader.getSystemClassLoader()
                             -> java.lang.ClassLoader.initSystemClassLoader()
                                (1) デフォルトのシステムクラスローダ(sun.misc.Launcher$AppClassLoader)を取得する.
                                    -> sun.misc.Launcher.getLauncher()  (<= sun.misc.Launcher$AppClassLoader.getAppClassLoader() で取得した値が返される)
                                (1) ユーザが指定したシステムクラスローダ("java.system.class.loader" システムプロパティに設定されたクラスローダ)を取得する.
                                    ("java.system.class.loader" が空の場合は, デフォルトのシステムクラスローダが返される)
                                    -> java.lang.SystemClassLoaderAction.run()

                      (2) メインクラス名を取得する.
                          -> * 指定されたクラスファイルがメインクラスである場合:
                               -> (引数で指定されたクラス名をそのまま使用)
                             * 指定された JAR ファイル内からメインクラスを探す場合: (= JAR 中の manifest ファイルから取得する場合)
                               -> sun.launcher.LauncherHelper.getMainClassFromJar()
         
                      (3) システムクラスローダの ClassLoader.loadClass() を呼び出し, メインクラスをロードする
                          (なお, オプショナルな第2引数である resolve 引数については false (resolve しない) で呼び出している)
                          -> sun.misc.Launcher$AppClassLoader.loadClass()
                             -> (See: [here](noYhX5khB4.html) for details)                             
         
                      (4) ロードしたクラスに main() メソッドがあり, static メソッドでかつ返値が void であることを確認.
                          -> sun.launcher.LauncherHelper.getMainMethod()
                             -> java.lang.Class.getMethod()
                                -> java.lang.Class.getMethod0()
                                   -> java.lang.Class.privateGetDeclaredMethods()
                                      -> java.lang.Class.getDeclaredMethods0()
                                         -> JVM_GetClassDeclaredMethods() (= java.lang.Class.getDeclaredMethods0())
                                            -> instanceKlass::link_class()
                                               -> (See: [here](no3059xqe.html) for details)
                                            -> Reflection::new_method()
                                   -> java.lang.Class.searchMethods()
                                   -> java.lang.Class.getMethod0()
                                      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### LoadMainClass()
See: [here](no2099VIJ.html) for details
### GetLauncherHelperClass()
See: [here](no20999gu.html) for details
### FindBootStrapClass()  (Solaris または Linux の場合)
See: [here](no2099jTW.html) for details
### FindBootStrapClass()  (Windows の場合)
See: [here](no2099KAR.html) for details
### JVM_FindClassFromBootLoader()
See: [here](no2099KHF.html) for details
### sun.launcher.LauncherHelper.checkAndLoadMain()
See: [here](no76256bi.html) for details
### java.lang.ClassLoader.getSystemClassLoader()
See: [here](no7625iCw.html) for details
### java.lang.ClassLoader.initSystemClassLoader()
See: [here](no76258dw.html) for details
### sun.misc.Launcher.getLauncher()
See: [here](no7625wpN.html) for details
### sun.misc.Launcher$AppClassLoader.getAppClassLoader()
See: [here](no7625-tm.html) for details
### java.lang.SystemClassLoaderAction.run()
See: [here](no7625-CD.html) for details
### sun.launcher.LauncherHelper.getMainClassFromJar()
See: [here](no7625MHc.html) for details
### sun.launcher.LauncherHelper.getMainMethod()
See: [here](no26814fag.html) for details
### java.lang.Class.getMethod()
See: [here](no268146vz.html) for details
### java.lang.Class.getMethod0()
See: [here](no26814gwD.html) for details
### java.lang.Class.privateGetDeclaredMethods()
See: [here](no268148_p.html) for details
### JVM_GetClassDeclaredMethods()
See: [here](no268148NS.html) for details
### Reflection::new_method()
(#Under Construction)

### java.lang.Class.searchMethods()
See: [here](no7517OBz.html) for details






