---
layout: default
title: ElfStringTable クラス 
---
[Top](../index.html)

#### ElfStringTable クラス 



---
## <a name="noBiJwykLx" id="noBiJwykLx">ElfStringTable</a>

### 概要(Summary)
ElfFile クラス内で使用される補助クラス (See: ElfFile).

Elf ファイル中の string table セクションを扱うためのもの.


```
    ((cite: hotspot/src/share/vm/utilities/elfStringTable.hpp))
    // The string table represents a string table section in an elf file.
    // Whenever there is enough memory, it will load whole string table as
    // one blob. Otherwise, it will load string from file when requested.
```


```
    ((cite: hotspot/src/share/vm/utilities/elfStringTable.hpp))
    class ElfStringTable: CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ElfFile オブジェクトの m_string_tables フィールドに(のみ)格納されている.

(正確には, このフィールドは ElfStringTable の線形リストを格納するフィールド.
ElfStringTable オブジェクトは m_next フィールドで次の ElfStringTable オブジェクトを指せる構造になっている.
その ElfFile オブジェクト内で生成した ElfStringTable オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
ElfFile::load_tables() 内で(のみ)生成されている.
そして, この関数は現在は ElfFile::ElfFile() 内で(のみ)呼び出されている.




### 詳細(Details)
See: [here](../doxygen/classElfStringTable.html) for details

---
