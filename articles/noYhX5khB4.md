---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理の開始点 ： その他のロード処理
---
[Up](no7ggAHQj6.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理の開始点 ： その他のロード処理

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### ロード後に発生するクラスファイル検証中に, (検証のために)他のクラスが必要になったとき.
(#Under Construction)

### クラスロードを伴う API が明示的に呼ばれたとき (java.lang.Class.forName(), java.lang.ClassLoader.loadClass(), Reflection APIs, etc)
* java.lang.Class.forName(String className)

```
java.lang.Class.forName(String className) (※)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> JVM_FindClassFromClassLoader()
      -> find_class_from_class_loader()
         -> SystemDictionary::resolve_or_fail()
            -> (See: [here](noIvSV0NZj.html) for details)
         -> Klass::initialize()
            -> (See: [here](no9AAGw84F.html) for details)

(※) なお, この関数はクラスローダーとして「現在のクラスを定義するクラスローダ」を使用.
```

* java.lang.Class.forName(String name, boolean initialize, ClassLoader loader)

```
java.lang.Class.forName(String name, boolean initialize, ClassLoader loader) (※)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> (同上)

(※) なお, この関数はクラスローダーとして「loader 引数で指定されたクラスローダ」を使用.
```

* java.lang.ClassLoader.loadClass()
  
  (以下は, デフォルトのシステムクラスローダ (sun.misc.Launcher$AppClassLoader) の場合)

```
sun.misc.Launcher$AppClassLoader.loadClass()
-> (1) package のチェックを行う.
       -> java.lang.SecurityManager.checkPackageAccess() 
   (2) スーパークラスの loadClass() に処理を委譲
       (URLClassLoader ではオーバーライドしていないため, java.lang.ClassLoader.loadClass() が呼び出される)
       -> java.lang.ClassLoader.loadClass()
          (1) 既にロード済みかどうかをチェック (ロード済みならそれを使う)
              -> java.lang.ClassLoader.findLoadedClass()
                 -> Java_java_lang_ClassLoader_findLoadedClass0() (= java.lang.ClassLoader.findLoadedClass0())
                    -> JVM_FindLoadedClass()
                       -> SystemDictionary::find_instance_or_array_klass()
                          -> Universe::typeArrayKlassObj()
                          -> Klass::array_klass_or_null()
                          -> SystemDictionary::find()

          (2) まず, 親に委譲してロードを試みる. 親がいなければ findBootstrapClassOrNull() でロードを試みる.
              (なお, システムクラスローダーの親は ExtClassLoader である模様)
              -> java.lang.ClassLoader.loadClass()
                 -> (同上)
              -> java.lang.ClassLoader.findBootstrapClassOrNull()
                 -> Java_java_lang_ClassLoader_findBootstrapClass() (= java.lang.ClassLoader.findBootstrapClass())
                    -> JVM_FindClassFromBootLoader()
                       -> SystemDictionary::resolve_or_null()
                          -> (See: [here](noIvSV0NZj.html) for details)

          (3) 以上で駄目なら, 自力でのロードを試みる. 
              (java.net.URLClassLoader.findClass() では, コンストラクタで指定された URL の下で指定のクラスファイルを見つけようとする.
              AppClassLoader が, System.getProperty("java.class.path") で取得したクラスパスを URLClassLoader のコンストラクタに渡しているため,
              クラスパス下にあるファイルが検出され, defineClass() でロードされる)
              -> java.net.URLClassLoader.findClass()
                 -> java.net.URLClassLoader.defineClass()
                    -> (1) (クラス名の中にパッケージ部分があれば) パッケージ名のチェックおよびパッケージオブジェクトの登録を行う
                           -> java.net.URLClassLoader.getAndVerifyPackage()
                           -> java.net.URLClassLoader.definePackage()
                              -> java.lang.ClassLoader.definePackage()
                           -> java.lang.ClassLoader.definePackage()
                       (1) クラスをロードする. (2通りのパスがあるがどちらも JVM_DefineClassWithSource() で合流)
                           -> * 指定のクラスファイル(Resource オブジェクト)から ByteBuffer が取得できなかった場合: 
                                -> java.lang.ClassLoader.defineClass(byte[] b, int off, int len)
                                   -> java.lang.ClassLoader.defineClass(String name, byte[] b, int off, int len)
                                      -> java.lang.ClassLoader.defineClass(String name, byte[] b, int off, int len, ProtectionDomain protectionDomain)
                                         -> Java_java_lang_ClassLoader_defineClass1()
                                            -> JVM_DefineClassWithSource()
                                               -> jvm_define_class_common()
                                                  -> SystemDictionary::resolve_from_stream()
                                                     -> (See: [here](noIvSV0NZj.html) for details)

                              * 〃 ByteBuffer が取得できた場合: 
                                -> java.lang.ClassLoader.defineClass(String name, java.nio.ByteBuffer b, ProtectionDomain protectionDomain)
                                   -> * direct ByteBufer でない場合:
                                        -> java.lang.ClassLoader.defineClass(String name, byte[] b, int off, int len, ProtectionDomain protectionDomain)
                                           -> (同上)
                                      * direct ByteBufer である場合:
                                        -> Java_java_lang_ClassLoader_defineClass2()
                                           -> JVM_DefineClassWithSource()
                                              -> (同上)
   
          (4) (引数で resolve まで行うよう指定されていれば) resolve 処理を行う.
              (が, 現状では何もしない)
              -> java.lang.ClassLoader.resolveClass()
                 -> Java_java_lang_ClassLoader_resolveClass0() (= java.lang.ClassLoader.resolveClass0())
                    -> JVM_ResolveClass()
                       -> (何もしない)
```

* Reflection APIs

  (#Under Construction)

* JNI の FindClass() 関数

  (See: [here](no1lbl8Grr.html) for details)

* JVMTI の RedefineClasses() 関数, 及び RetransformClasses() 関数

  (See: [here](no2935-Vj.html) for details)


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.Class.forName(String className)
See: [here](no7954aRT.html) for details
### Java_java_lang_Class_forName0()
See: [here](no7954pWz.html) for details
### JVM_FindClassFromClassLoader()
See: [here](no79545GT.html) for details
### find_class_from_class_loader()
(#Under Construction)
See: [here](no3059EQr.html) for details
### java.lang.Class.forName(String name, boolean initialize, ClassLoader loader)
See: [here](no79540sT.html) for details

### sun.misc.Launcher$AppClassLoader.loadClass()
See: [here](no7625zlu.html) for details
### java.lang.SecurityManager.checkPackageAccess()
(#Under Construction)
See: [here](no7625aSp.html) for details
### java.lang.ClassLoader.loadClass()
See: [here](no7625bTw.html) for details
### java.lang.ClassLoader.findLoadedClass()
See: [here](no76251uw.html) for details
### Java_java_lang_ClassLoader_findLoadedClass0()
See: [here](no7625bhY.html) for details
### JVM_FindLoadedClass()
See: [here](no7625psl.html) for details
### SystemDictionary::find_instance_or_array_klass()
See: [here](no75172DR.html) for details
### SystemDictionary::find()
See: [here](no7517fyZ.html) for details
### java.lang.ClassLoader.findBootstrapClassOrNull()
See: [here](no7625ruz.html) for details
### Java_java_lang_ClassLoader_findBootstrapClass()
See: [here](no7625r1n.html) for details
### java.net.URLClassLoader.findClass()
See: [here](no7625DWO.html) for details
### java.net.URLClassLoader.defineClass()
See: [here](no7625Tqd.html) for details
### java.net.URLClassLoader.getAndVerifyPackage()
(#Under Construction)

### java.net.URLClassLoader.definePackage()
(#Under Construction)

### java.lang.ClassLoader.definePackage()
(#Under Construction)

### java.lang.ClassLoader.defineClass(byte[] b, int off, int len)
See: [here](no76257eT.html) for details
### java.lang.ClassLoader.defineClass(String name, byte[] b, int off, int len)
See: [here](no76257lH.html) for details
### java.lang.ClassLoader.defineClass(String name, byte[] b, int off, int len, ProtectionDomain protectionDomain)
(#Under Construction)
See: [here](no7625Wty.html) for details
### Java_java_lang_ClassLoader_defineClass1()
See: [here](no76258tC.html) for details
### JVM_DefineClassWithSource()
See: [here](no7625KAE.html) for details
### jvm_define_class_common()
See: [here](no7625kbE.html) for details
### SystemDictionary::resolve_from_stream()
See: [here](no7625MJG.html) for details
### SystemDictionary::is_parallelCapable()
(#Under Construction)
See: [here](no7625pnb.html) for details
### SystemDictionary::find_or_define_instance_class()
See: [here](no7625QGu.html) for details
### SystemDictionary::define_instance_class()
(#Under Construction)
See: [here](no7625r3R.html) for details
### java.lang.ClassLoader.addClass()
See: [here](no7625TXr.html) for details
### SystemDictionary::add_to_hierarchy()
(#Under Construction)
See: [here](no26814kOG.html) for details
### Universe::flush_dependents_on()
(#Under Construction)


### SystemDictionary::update_dictionary()
See: [here](no26814_jZ.html) for details
### Dictionary::add_klass()
See: [here](no26814O3h.html) for details
### instanceKlass::eager_initialize()
(#Under Construction)
See: [here](no26814X2X.html) for details
### java.lang.ClassLoader.defineClass(String name, java.nio.ByteBuffer b, ProtectionDomain protectionDomain)
(#Under Construction)
See: [here](no26814B0P.html) for details
### Java_java_lang_ClassLoader_defineClass2()
See: [here](no268143e2.html) for details
### java.lang.ClassLoader.resolveClass()
See: [here](no7517wIZ.html) for details
### Java_java_lang_ClassLoader_resolveClass0()
See: [here](no7517iEA.html) for details
### JVM_ResolveClass()
See: [here](no7517X1T.html) for details





