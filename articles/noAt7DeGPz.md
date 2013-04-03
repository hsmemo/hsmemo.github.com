---
layout: default
title: コマンドラインオプションを処理するためのクラス (SystemProperty, AgentLibrary, AgentLibraryList, Arguments, 及びそれらの補助クラス(SysClassPath))
---
[Top](../index.html)

#### コマンドラインオプションを処理するためのクラス (SystemProperty, AgentLibrary, AgentLibraryList, Arguments, 及びそれらの補助クラス(SysClassPath))

これらは, ユーザーが指定した HotSpot のコマンドラインオプションを管理するためのクラス
(それに関連して JVMTI エージェントの情報も管理している).


### クラス一覧(class list)

  * [SystemProperty](#nozqW8018B)
  * [AgentLibrary](#noiyNJ1Rqx)
  * [AgentLibraryList](#no8fPODMDI)
  * [Arguments](#noE60BAGyj)
  * [SysClassPath](#no729k7a-j)


---
## <a name="nozqW8018B" id="nozqW8018B">SystemProperty</a>

### 概要(Summary)
HotSpot のシステムプロパティ (= java.lang.System.getProperty() メソッドで取得できる情報) を管理するためのクラス.

1つの SystemProperty オブジェクトが 1つのシステムプロパティに対応する.


```
    ((cite: hotspot/src/share/vm/runtime/arguments.hpp))
    // Element describing System and User (-Dkey=value flags) defined property.
    
    class SystemProperty: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Arguments クラスの _system_properties フィールド (static フィールド) に(のみ)格納されている.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている.

* Arguments::init_system_properties()
* Arguments::init_version_specific_system_properties()
* Arguments::PropertyList_add(SystemProperty** plist, const char* k, char* v)




### 詳細(Details)
See: [here](../doxygen/classSystemProperty.html) for details

---
## <a name="noiyNJ1Rqx" id="noiyNJ1Rqx">AgentLibrary</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能用のクラス).

HotSpot がロードした JVMTI エージェントの情報を表す.
1つの AgentLibrary オブジェクトが 1つの JVMTI エージェントに対応する.


```
    ((cite: hotspot/src/share/vm/runtime/arguments.hpp))
    // For use by -agentlib, -agentpath and -Xrun
    class AgentLibrary : public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
各 AgentLibraryList オブジェクトの _first フィールドおよび _last フィールドに(のみ)格納されている.

(正確には, これらのフィールドは AgentLibrary の線形リスト(の先頭と最後)を格納するフィールド.
AgentLibrary オブジェクトは _next フィールドで次の AgentLibrary オブジェクトを指せる構造になっている.
その AgentLibraryList オブジェクト用の AgentLibrary オブジェクトは全てこの線形リストに格納されている)


#### 生成箇所(where its instances are created)
以下のファクトリメソッドが用意されており, その中で(のみ)生成されている.

* Arguments::add_init_library()
  
  -Xrun オプションで指定された JVMTI エージェントを表す AgentLibrary オブジェクトが生成され, 
  Arguments::_libraryList につながれる.

* Arguments::add_init_agent()
  
  -javaagent オプションで指定された JVMTI エージェントの情報を _agentList につなぐ.

* Arguments::add_loaded_agent()
  
  Dynamic Attach 機能で指定された JVMTI エージェントの情報を _agentList につなぐ.




### 詳細(Details)
See: [here](../doxygen/classAgentLibrary.html) for details

---
## <a name="no8fPODMDI" id="no8fPODMDI">AgentLibraryList</a>

### 概要(Summary)
保守運用機能のためのクラス (JVMTI 機能用のクラス).

AgentLibrary オブジェクトを束ねておくためのコンテナクラス.


```
    ((cite: hotspot/src/share/vm/runtime/arguments.hpp))
    // maintain an order of entry list of AgentLibrary
    class AgentLibraryList VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
以下の箇所に(のみ)格納されている.

* Arguments クラスの _libraryList フィールド (static フィールド)
  
  -Xrun オプションで指定された JVMTI エージェントを格納する.

* Arguments クラスの _agentList フィールド (static フィールド)
  
  -agentlib オプション, -agentpath オプション, 及び Dynamic Attach 機能で動的にロードされた JVMTI エージェントを格納する.

#### 生成箇所(where its instances are created)
(Arguments クラスの _libraryList フィールドおよび _agentList フィールドは, ポインタ型ではなく実体なので,
 Arguments オブジェクトの生成時に一緒に生成される)




### 詳細(Details)
See: [here](../doxygen/classAgentLibraryList.html) for details

---
## <a name="noE60BAGyj" id="noE60BAGyj">Arguments</a>

### 概要(Summary)
ユーザーが指定した HotSpot のコマンドラインオプションを管理するクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

コマンドラインオプションをパースするメソッドや, パースした各オプション値へのアクセサメソッド等を提供している.


```
    ((cite: hotspot/src/share/vm/runtime/arguments.hpp))
    class Arguments : AllStatic {
```

### 使われ方(Usage)
Arguments::parse() で, コマンドラインオプションのパースが行われる.

パースした各オプションの値は, HotSpot 内の様々な箇所で参照されている (#TODO).




### 詳細(Details)
See: [here](../doxygen/classArguments.html) for details

---
## <a name="no729k7a-j" id="no729k7a-j">SysClassPath</a>

### 概要(Summary)
Arguments クラス内で使用される補助クラス.

HotSpot の system class path 情報 (boot class path 情報) を作成するための補助クラス(StackObjクラス).

なお, system class path 情報は以下の 4つの情報に基づいて作成される.

  * prefix (from -Xbootclasspath/p:...)
  * endorsed (the expansion of -Djava.endorsed.dirs=...)
  * base (from os::get_system_properties() or -Xbootclasspath=)
  * suffix (from -Xbootclasspath/a:...)

(なおコメントによると, AllStatic クラスにしてもいいがコマンドラインオプションのパースが終わった後は使われない, とのこと)


```
    ((cite: hotspot/src/share/vm/runtime/arguments.cpp))
    // Constructs the system class path (aka boot class path) from the following
    // components, in order:
    //
    //     prefix           // from -Xbootclasspath/p:...
    //     endorsed         // the expansion of -Djava.endorsed.dirs=...
    //     base             // from os::get_system_properties() or -Xbootclasspath=
    //     suffix           // from -Xbootclasspath/a:...
    //
    // java.endorsed.dirs is a list of directories; any jar or zip files in the
    // directories are added to the sysclasspath just before the base.
    //
    // This could be AllStatic, but it isn't needed after argument processing is
    // complete.
    class SysClassPath: public StackObj {
```

### 使われ方(Usage)
Arguments::parse_vm_init_args() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classSysClassPath.html) for details

---
