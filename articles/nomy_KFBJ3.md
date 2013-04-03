---
layout: default
title: ElfSymbolTable クラス 
---
[Top](../index.html)

#### ElfSymbolTable クラス 



---
## <a name="nok3eRwCON" id="nok3eRwCON">ElfSymbolTable</a>

### 概要(Summary)
ElfFile クラス内で使用される補助クラス (See: ElfFile).

Elf ファイルの symbol セクションを扱うためのもの.


```
    ((cite: hotspot/src/share/vm/utilities/elfSymbolTable.hpp))
    /*
     * symbol table object represents a symbol section in an elf file.
     * Whenever possible, it will load all symbols from the corresponding section
     * of the elf file into memory. Otherwise, it will walk the section in file
     * to look up the symbol that nearest the given address.
     */
    class ElfSymbolTable: public CHeapObj {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
ElfFile オブジェクトの m_symbol_tables フィールドに(のみ)格納されている.

(正確には, このフィールドは ElfSymbolTable の線形リストを格納するフィールド.
ElfSymbolTable オブジェクトは m_next フィールドで次の ElfSymbolTable オブジェクトを指せる構造になっている.
その ElfFile オブジェクト内で生成した ElfSymbolTable オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
ElfFile::load_tables() 内で(のみ)生成されている.
そして, この関数は現在は ElfFile::ElfFile() 内で(のみ)呼び出されている.




### 詳細(Details)
See: [here](../doxygen/classElfSymbolTable.html) for details

---
