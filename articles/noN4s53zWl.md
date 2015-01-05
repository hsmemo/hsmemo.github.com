---
layout: default
title: SystemDictionary クラス関連のクラス (SystemDictionary, SystemDictionaryHandles, 及びそれらの補助クラス(ClassStatistics, MethodStatistics))
---
[Top](../index.html)

#### SystemDictionary クラス関連のクラス (SystemDictionary, SystemDictionaryHandles, 及びそれらの補助クラス(ClassStatistics, MethodStatistics))

これらは, クラスのロード処理用, 及びロードしたクラス情報の管理用のクラス (See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details).


### クラス一覧(class list)

  * [SystemDictionary](#nog_UUI00h)
  * [SystemDictionaryHandles](#nom57KMojW)
  * [ClassStatistics](#noEyGJKoue)
  * [MethodStatistics](#nof0z_BnHn)


---
## <a name="nog_UUI00h" id="nog_UUI00h">SystemDictionary</a>

### 概要(Summary)
クラスのロード処理を行う関数, 及びロードしたクラスを管理するための関数を納めた名前空間(AllStatic クラス).


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
    class SystemDictionary : AllStatic {
```

内部的には, ロード済みのJavaクラスを記憶しておくハッシュテーブルになっている.
クラス名(Symbol)とクラスローダ(oop)をキーとして, クラス(klassOop)を返す
(なお, クラスローダとして default VM class loader を指定したい場合は NULL をキーとすればいい模様).
(See: [here](no7882m2Z.html) and [here](no7882ALm.html) for details)


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
    // The system dictionary stores all loaded classes and maps:
    //
    //   [class name,class loader] -> class   i.e.  [Symbol*,oop] -> klassOop
    //
    // Classes are loaded lazily. The default VM class loader is
    // represented as NULL.
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

### 内部構造(Internal structure)
実際にハッシュテーブルとしての機能を実装しているのは Dictionary クラス (See: Dictionary). 
SystemDictionary はそれを static フィールドに保持して使用しているだけ.

なお, SystemDictionary は以下の 2つの Dictionary オブジェクトを使用している.

 * ロードしたクラス用
 * shared archive から取得したクラス用 (なお, shared archive とは Class Data Sharing (CDS) の領域のこと)


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
      // Hashtable holding loaded classes.
      static Dictionary*            _dictionary;
    ...
      // Hashtable holding classes from the shared archive.
      static Dictionary*             _shared_dictionary;
```




### 詳細(Details)
See: [here](../doxygen/classSystemDictionary.html) for details

---
## <a name="nom57KMojW" id="nom57KMojW">SystemDictionaryHandles</a>

### 概要(Summary)
SystemDictionary クラスのラッパークラス.

標準ライブラリ内のクラスへの簡単なアクセスを提供するクラス.
SystemDictionary との違いは, 返り値の型が klassOop ではなく klassHandle である点
(SystemDictionaryHandles が klassOop を Handle 化してくれるので, 自分で Handle 化する手間が省ける).


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.hpp))
    class SystemDictionaryHandles : AllStatic {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
各 Java クラスに対応したメソッドが用意されており, それを呼び出すことで Handle 化された klassOop (KlassHandle) が得られる
(例: SystemDictionaryHandles::Object_klass(), 
 SystemDictionaryHandles::Class_klass(), SystemDictionaryHandles::MethodHandle_klass(), etc).

#### 使用箇所(where its instances are used)
(現状ではほとんど MethodHandle 関連の箇所でしか使われていないような... #TODO)




### 詳細(Details)
See: [here](../doxygen/classSystemDictionaryHandles.html) for details

---
## <a name="noEyGJKoue" id="noEyGJKoue">ClassStatistics</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

クラスに関する統計情報出力用の関数を納めた名前空間(AllStatic クラス).
(e.g. ロード済みのクラスの個数, それらが消費しているメモリ量, ロード済みのメソッド数, それらが消費しているメモリ量, etc)


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.cpp))
    #ifndef PRODUCT
    
    // statistics code
    class ClassStatistics: AllStatic {
```

### 使われ方(Usage)
SystemDictionary::print_class_statistics() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
print_statistics()
-&gt; SystemDictionary::print_class_statistics()
   -&gt; ClassStatistics::print()
</pre></div>

なお, このクラスは (デバッグ時であることに加えて) PrintClassStatistics オプションが指定されている場合にしか使用されない.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
    void print_statistics() {
    ...
      if (PrintClassStatistics) {
        SystemDictionary::print_class_statistics();
      }
```

#### 参考(for your information): SystemDictionary::print_class_statistics()
See: [here](no2747eg1.html) for details
### 内部構造(Internal structure)
実行中に情報を取っておくのではなく, 呼び出された時点になってから SystemDictionary 内を調べている模様.
最終的には ClassStatistics::print() で出力が行われる.

#### 参考(for your information): ClassStatistics::print()
See: [here](no2747QqE.html) for details



### 詳細(Details)
See: [here](../doxygen/classClassStatistics.html) for details

---
## <a name="nof0z_BnHn" id="nof0z_BnHn">MethodStatistics</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).

メソッドに関する統計情報出力用の関数を納めた名前空間(AllStatic クラス).
(e.g. 各 attribute (final, static, synchronized, etc) がついているメソッドがそれぞれ何個あったか, 
 引数の個数に関するヒストグラム情報(引数の個数が 1個,2個,3個,... のメソッドがそれぞれいくら存在するか), 
 全メソッド中での各bytecodeの登場比率はいくらか, etc)


```cpp
    ((cite: hotspot/src/share/vm/classfile/systemDictionary.cpp))
    class MethodStatistics: AllStatic {
```

### 使われ方(Usage)
SystemDictionary::print_method_statistics() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

<div class="flow-abst"><pre>
print_statistics()
-&gt; SystemDictionary::print_method_statistics()
   -&gt; MethodStatistics::print()
</pre></div>

なお, このクラスは (デバッグ時であることに加えて) PrintMethodStatistics オプションが指定されている場合にしか使用されない.


```cpp
    ((cite: hotspot/src/share/vm/runtime/java.cpp))
      if (PrintMethodStatistics) {
        SystemDictionary::print_method_statistics();
      }
```

#### 参考(for your information): SystemDictionary::print_method_statistics()
See: [here](no2747d0K.html) for details
### 内部構造(Internal structure)
実行中に情報を取っておくのではなく, 呼び出された時点になってから SystemDictionary 内を調べている模様.
最終的には MethodStatistics::print() で出力が行われる.

#### 参考(for your information): MethodStatistics::print()
See: [here](no27473IX.html) for details



### 詳細(Details)
See: [here](../doxygen/classMethodStatistics.html) for details

---
