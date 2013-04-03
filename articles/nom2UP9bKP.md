---
layout: default
title: Abstract_VM_Version クラス 
---
[Top](../index.html)

#### Abstract_VM_Version クラス 



---
## <a name="no28GEXyY4" id="no28GEXyY4">Abstract_VM_Version</a>

### 概要(Summary)
HotSpot の製品情報を納めた名前空間(AllStatic クラス).

例えば次のような情報を格納している.

* メジャーバージョン情報
* マイナーバージョン情報
* ビルドナンバー情報
* ベンダー情報
* etc


```
    ((cite: hotspot/src/share/vm/runtime/vm_version.hpp))
    // VM_Version provides information about the VM.
    
    class Abstract_VM_Version: AllStatic {
```

なお, "Abstract_" というクラス名だが abstract class ではない 
(というか AllStatic なので関係ない. 実際このクラスのメソッドは使用されている).

### 内部構造(Internal structure)
ビルド時に, 以下のマクロ定数が定義されている必要がある (内部的にこれらの情報を使用する).

* HOTSPOT_RELEASE_VERSION
* JRE_RELEASE_VERSION
* HOTSPOT_BUILD_TARGET
* HOTSPOT_VM_DISTRO


```
    ((cite: hotspot/src/share/vm/runtime/vm_version.cpp))
    #ifndef HOTSPOT_RELEASE_VERSION
      #error HOTSPOT_RELEASE_VERSION must be defined
    #endif
    #ifndef JRE_RELEASE_VERSION
      #error JRE_RELEASE_VERSION must be defined
    #endif
    #ifndef HOTSPOT_BUILD_TARGET
      #error HOTSPOT_BUILD_TARGET must be defined
    #endif
```


```
    ((cite: hotspot/src/share/vm/runtime/vm_version.cpp))
    #ifndef HOTSPOT_VM_DISTRO
      #error HOTSPOT_VM_DISTRO must be defined
    #endif
```

また上記以外にも製品情報を定義するためのマクロ定数が多数存在している.

### 備考(Notes)
ベンダー情報を変更するには, VENDOR というマクロ定数を定義すればいい (このマクロにベンダー名を設定する).

#### 参考(for your information): Abstract_VM_Version::vm_vendor()
See: [here](no3420Y-h.html) for details



### 詳細(Details)
See: [here](../doxygen/classAbstract__VM__Version.html) for details

---
