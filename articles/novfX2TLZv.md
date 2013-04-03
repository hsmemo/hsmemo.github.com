---
layout: default
title: ブートストラップ・クラスローダ関連のクラス (MetaIndex, ClassPathEntry, ClassPathDirEntry, ClassPathZipEntry, LazyClassPathEntry, ClassLoader, PerfClassTraceTime, 及びそれらの補助クラス(PackageInfo, PackageHashtable))
---
[Top](../index.html)

#### ブートストラップ・クラスローダ関連のクラス (MetaIndex, ClassPathEntry, ClassPathDirEntry, ClassPathZipEntry, LazyClassPathEntry, ClassLoader, PerfClassTraceTime, 及びそれらの補助クラス(PackageInfo, PackageHashtable))

これらは, クラスローディング処理用のクラス.
より具体的に言うと, HotSpot のブートストラップ・クラスローダ(bootstrap class loader)を実現するためのクラス
(See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).


### クラス一覧(class list)

  * [ClassLoader](#nolQspE5hm)
  * [ClassPathEntry](#noQtARf7kD)
  * [ClassPathDirEntry](#nop6oduQ8-)
  * [ClassPathZipEntry](#noeF66RgqZ)
  * [LazyClassPathEntry](#nozW67_Ltn)
  * [MetaIndex](#no1eiFh_NB)
  * [PerfClassTraceTime](#noMnCvVSyr)
  * [PackageHashtable](#noNOfikWoq)
  * [PackageInfo](#noMC15RpnO)


---
## <a name="nolQspE5hm" id="nolQspE5hm">ClassLoader</a>

### 概要(Summary)
ブートストラップ・クラスローダに関する機能を納めた名前空間(AllStatic クラス).


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    class ClassLoader: AllStatic {
```




### 詳細(Details)
See: [here](../doxygen/classClassLoader.html) for details

---
## <a name="noQtARf7kD" id="noQtARf7kD">ClassPathEntry</a>

### 概要(Summary)
ClassLoader クラス用の補助クラス.

ロード対象のクラスファイルのパスを表すクラス(の基底クラス).
1つの ClassPathEntry オブジェクトが 1つのディレクトリもしくは 1つの zip ファイルに対応する.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    // Class path entry (directory or zip file)
    
    class ClassPathEntry: public CHeapObj {
```




### 詳細(Details)
See: [here](../doxygen/classClassPathEntry.html) for details

---
## <a name="nop6oduQ8-" id="nop6oduQ8-">ClassPathDirEntry</a>

### 概要(Summary)
ClassPathEntry クラスの具象サブクラスの1つ.

このクラスは, パスが「クラスファイルの入ったディレクトリ」である場合用.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    class ClassPathDirEntry: public ClassPathEntry {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ClassLoader::create_class_path_entry() 内で(のみ)生成されている (See: [here](no7882afy.html) for details).




### 詳細(Details)
See: [here](../doxygen/classClassPathDirEntry.html) for details

---
## <a name="noeF66RgqZ" id="noeF66RgqZ">ClassPathZipEntry</a>

### 概要(Summary)
ClassPathEntry クラスの具象サブクラスの1つ.

このクラスは, パスが「クラスファイルの入った jar ファイル」である場合用.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    class ClassPathZipEntry: public ClassPathEntry {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* ClassLoader::create_class_path_entry() (See: [here](no7882afy.html) for details)

* ClassLoader::create_class_path_zip_entry() (See: [here](no7882MpB.html) for details)




### 詳細(Details)
See: [here](../doxygen/classClassPathZipEntry.html) for details

---
## <a name="nozW67_Ltn" id="nozW67_Ltn">LazyClassPathEntry</a>

### 概要(Summary)
ClassPathEntry クラスの具象サブクラスの1つ.

このクラスは, 標準ライブラリの jar ファイルを遅延ロードする場合用.
LazyBootClassLoader オプションが指定されている場合に使用される (See: LazyBootClassLoader オプション).


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    // For lazier loading of boot class path entries
    class LazyClassPathEntry: public ClassPathEntry {
```

### 使われ方(Usage)
#### 生成箇所(where its instances are created)
ClassLoader::create_class_path_entry() 内で(のみ)生成されている (See: [here](no7882afy.html) for details).




### 詳細(Details)
See: [here](../doxygen/classLazyClassPathEntry.html) for details

---
## <a name="no1eiFh_NB" id="no1eiFh_NB">MetaIndex</a>

### 概要(Summary)
LazyClassPathEntry クラス用の補助クラス.

起動時に読んだ meta-index ファイルの中身を格納しておくためのクラス (See: LazyBootClassLoader オプション).
不必要な jar ファイルのロードを防止するために使われる.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    // Meta-index (optional, to be able to skip opening boot classpath jar files)
    class MetaIndex: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 LazyClassPathEntry オブジェクトの _meta_index フィールドに(のみ)格納されている.

#### 生成箇所(where its instances are created)
ClassLoader::setup_meta_index() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classMetaIndex.html) for details

---
## <a name="noMnCvVSyr" id="noMnCvVSyr">PerfClassTraceTime</a>

### 概要(Summary)
クラスローディングにかかった処理時間(あるいは関連する処理時間)を記録するためのユーティリティ・クラス.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
    // PerfClassTraceTime is used to measure time for class loading related events.
    // This class tracks cumulative time and exclusive time for specific event types.
    // During the execution of one event, other event types (e.g. class loading and
    // resolution) as well as recursive calls of the same event type could happen.
    // Only one elapsed timer (cumulative) and one thread-local self timer (exclusive)
    // (i.e. only one event type) are active at a time even multiple PerfClassTraceTime
    // instances have been created as multiple events are happening.
    class PerfClassTraceTime {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ClassFileParser::parseClassFile()
* ClassLoader::load_classfile()
* SystemDictionary::load_instance_class()
* instanceKlass::link_class_impl()
* jvm_define_class_common()

### 内部構造(Internal structure)
コンストラクタで計測が開始され, デストラクタで計測が終了する.

#### 参考(for your information): PerfClassTraceTime::PerfClassTraceTime()
See: [here](no17119x1Z.html) for details
#### 参考(for your information): PerfClassTraceTime::initialize()
See: [here](no17119krT.html) for details
#### 参考(for your information): PerfClassTraceTime::~PerfClassTraceTime()
See: [here](no17119XhN.html) for details



### 詳細(Details)
See: [here](../doxygen/classPerfClassTraceTime.html) for details

---
## <a name="noNOfikWoq" id="noNOfikWoq">PackageHashtable</a>

### 概要(Summary)
ClassLoader クラス内で使用される補助クラス.

ロード済みの java パッケージを覚えておくためのクラス
(java.lang.Package クラスの getPackage() メソッドや getPackages() メソッドを実現するためのクラス).

内部はハッシュテーブルとして実装されている.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.cpp))
    class PackageHashtable : public BasicHashtable {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ClassLoader クラスの _package_hash_table フィールド (static フィールド) に(のみ)格納されている.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.hpp))
      // Hash table used to keep track of loaded packages
      static PackageHashtable* _package_hash_table;
```

#### 使用箇所(where its instances are used)
* ClassLoader::load_classfile() でロードされるたびに add_package() でハッシュに追加されていく.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.cpp))
    instanceKlassHandle ClassLoader::load_classfile(Symbol* h_name, TRAPS) {
    ...
        instanceKlassHandle result = parser.parseClassFile(h_name,
                                                           class_loader,
                                                           protection_domain,
                                                           parsed_name,
                                                           false,
                                                           CHECK_(h));
    
        // add to package table
        if (add_package(name, classpath_index, THREAD)) {
    ...
```

* 蓄えられたパッケージ情報は, Java_java_lang_Package_getSystemPackage0() や
  Java_java_lang_Package_getSystemPackages0() で参照される.
  
  (Java_java_lang_Package_getSystemPackages0() からは JVM_GetSystemPackages() が呼び出される.
  そこから ClassLoader::get_system_packages() が呼び出され, 
  最終的に ClassLoader::_package_hash_table 内のデータがコピーされてリターンされる.)


```
    ((cite: jdk/src/share/native/java/lang/Package.c))
    JNIEXPORT jstring JNICALL
    Java_java_lang_Package_getSystemPackage0(JNIEnv *env, jclass cls, jstring str)
    {
        return JVM_GetSystemPackage(env, str);
    }
    
    JNIEXPORT jobject JNICALL
    Java_java_lang_Package_getSystemPackages0(JNIEnv *env, jclass cls)
    {
        return JVM_GetSystemPackages(env);
    }
```


```
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    JVM_ENTRY(jobjectArray, JVM_GetSystemPackages(JNIEnv *env))
      JVMWrapper("JVM_GetSystemPackages");
      JvmtiVMObjectAllocEventCollector oam;
      objArrayOop result = ClassLoader::get_system_packages(CHECK_NULL);
      return (jobjectArray) JNIHandles::make_local(result);
    JVM_END
```

### 内部構造(Internal structure)
実際のパッケージ情報は PackageInfo オブジェクト内に格納されている (See: PackageInfo).




### 詳細(Details)
See: [here](../doxygen/classPackageHashtable.html) for details

---
## <a name="noMC15RpnO" id="noMC15RpnO">PackageInfo</a>

### 概要(Summary)
PackageHashtable クラス内で使用される補助クラス.

PackageHashtable オブジェクト内に格納されるハッシュテーブル・エントリ.
1つの PackageInfo オブジェクトが 1つの java パッケージに対応する.

BasicHashtableEntry のサブクラスになっており, 
パッケージ名をキーとしてハッシュから取得できるようになっている.


```
    ((cite: hotspot/src/share/vm/classfile/classLoader.cpp))
    // PackageInfo data exists in order to support the java.lang.Package
    // class.  A Package object provides information about a java package
    // (version, vendor, etc.) which originates in the manifest of the jar
    // file supplying the package.  For application classes, the ClassLoader
    // object takes care of this.
    
    // For system (boot) classes, the Java code in the Package class needs
    // to be able to identify which source jar file contained the boot
    // class, so that it can extract the manifest from it.  This table
    // identifies java packages with jar files in the boot classpath.
    
    // Because the boot classpath cannot change, the classpath index is
    // sufficient to identify the source jar file or directory.  (Since
    // directories have no manifests, the directory name is not required,
    // but is available.)
    
    // When using sharing -- the pathnames of entries in the boot classpath
    // may not be the same at runtime as they were when the archive was
    // created (NFS, Samba, etc.).  The actual files and directories named
    // in the classpath must be the same files, in the same order, even
    // though the exact name is not the same.
    
    class PackageInfo: public BasicHashtableEntry {
```

### 使われ方(Usage)
PackageHashtable::new_entry() 内で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classPackageInfo.html) for details

---
