---
layout: default
title: ElfFile クラス 
---
[Top](../index.html)

#### ElfFile クラス 



---
## <a name="nobH4w9tZm" id="nobH4w9tZm">ElfFile</a>

### 概要(Summary)
Decoder クラス内で使用される補助クラス (See: Decoder).

ELF ファイルの中身をパースするクラス.
主に, エラー発生時に指定されたアドレス付近のシンボルを探し出すために使用される.

(なおコメントによると, エラー時に使用されるクラスなので上手く動かないことも想定して実装している, とのこと)


```cpp
    ((cite: hotspot/src/share/vm/utilities/elfFile.hpp))
    // On Solaris/Linux platforms, libjvm.so does contain all private symbols.
    // ElfFile is basically an elf file parser, which can lookup the symbol
    // that is the nearest to the given address.
    // Beware, this code is called from vm error reporting code, when vm is already
    // in "error" state, so there are scenarios, lookup will fail. We want this
    // part of code to be very defensive, and bait out if anything went wrong.
    
    class ElfFile: public CHeapObj {
```

なお, このクラスは ELF を使用しない Windows 環境では (当然ながら) 使用されない.

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
Decoder クラスの _opened_elf_files フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは ElfFile の線形リストを格納するフィールド.
ElfFile オブジェクトは m_next フィールドで次の ElfFile オブジェクトを指せる構造になっており,
一度生成した ElfFile オブジェクトは全てこのフィールドの線形リストに格納されている)

#### 生成箇所(where its instances are created)
Decoder::get_elf_file() 内で(のみ)生成されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* VMError による処理 (print_stack_trace)

  VMError::print_stack_trace()
  -> frame::print_on_error()
     -> print_C_frame()
        -> os::dll_address_to_function_name()  (<= Solaris 版 or Linux 版)
           -> Decoder::decode()
              -> Decoder::get_elf_file()

* FlatProfilerTask による処理

  FlatProfilerTask::task()
  -> FlatProfiler::record_vm_tick()
     -> os::dll_address_to_function_name()  (<= Solaris 版 or Linux 版)
        -> (同上)
```




### 詳細(Details)
See: [here](../doxygen/classElfFile.html) for details

---
