---
layout: default
title: VM_Version クラス (VM_Version, 及びその補助クラス(VM_Version_StubGenerator))
---
[Top](../index.html)

#### VM_Version クラス (VM_Version, 及びその補助クラス(VM_Version_StubGenerator))

これらは, 稼働しているプロセッサの情報を取得するためのクラス.


### クラス一覧(class list)

  * [VM_Version](#no9q9p0UzU)
  * [VM_Version_StubGenerator](#nol6V7BRjl)


---
## <a name="no9q9p0UzU" id="no9q9p0UzU">VM_Version</a>

### 概要(Summary)
Abstract_VM_Version クラスの具象サブクラス (なお, 現在は (x86 用としては) このクラスが唯一のサブクラス)
(See: Abstract_VM_Version).

稼働しているプロセッサの情報を取得するためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

取得できる情報は, 例えば, ベンダー(Intel/AMD), プロセッサのモデル(386/486/Pentium/...), コア数, キャッシュラインサイズ, 各拡張命令への対応の有無, 等.


```
    ((cite: hotspot/src/cpu/x86/vm/vm_version_x86.hpp))
    class VM_Version : public Abstract_VM_Version {
```




### 詳細(Details)
See: [here](../doxygen/classVM__Version.html) for details

---
## <a name="nol6V7BRjl" id="nol6V7BRjl">VM_Version_StubGenerator</a>

### 概要(Summary)
VM_Version クラス内で使用される補助クラス(StubCodeGenerator クラス).

「CPUID 情報を取得する」ためのコードを実行時に生成する.


```
    ((cite: hotspot/src/cpu/x86/vm/vm_version_x86.cpp))
    class VM_Version_StubGenerator: public StubCodeGenerator {
```

### 使われ方(Usage)
VM_Version::initialize() 内で(のみ)使用されている.

### 内部構造(Internal structure)
内部には, (コンストラクタを除けば) 以下のメソッド(のみ)が定義されている.

* VM_Version_StubGenerator::generate_getPsrInfo() : 「CPUID 情報を取得する」コードを生成する
  
### 備考(Notes)
生成したコードは VM_Version::get_processor_features() で使用される.




### 詳細(Details)
See: [here](../doxygen/classVM__Version__StubGenerator.html) for details

---
