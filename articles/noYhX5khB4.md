---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理(Loading) の開始契機 ： その他
---
[Up](no7ggAHQj6.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理(Loading) の開始契機 ： その他

--- 
## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### ロード後に発生するクラスファイル検証中に, (検証のために)他のクラスが必要になったとき.
(#Under Construction)

### クラスロードを伴う API が明示的に呼ばれたとき (Class.forName(), ClassLoader.loadClass(), Reflection APIs, etc)
* Class.forName(String className)

```
java.lang.Class.forName(String className)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> JVM_FindClassFromClassLoader()
      -> find_class_from_class_loader()
         -> SystemDictionary::resolve_or_fail()
            -> (See: [here](notXYWwprj.html) for details)
         -> Klass::initialize()
            -> (See: [here](no9AAGw84F.html) for details)
```

* Class.forName(String name, boolean initialize, ClassLoader loader)

```
java.lang.Class.forName(String name, boolean initialize, ClassLoader loader)
-> Java_java_lang_Class_forName0()  (= java.lang.Class.forName0())
   -> (同上)
```

* ClassLoader.loadClass()

  (See: [here](notXYWwprj.html) for details)

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






