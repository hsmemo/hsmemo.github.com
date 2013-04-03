---
layout: default
title: Decoder クラス 
---
[Top](../index.html)

#### Decoder クラス 



---
## <a name="noIrGGvAZa" id="noIrGGvAZa">Decoder</a>

### 概要(Summary)
トラブルシューティング用のクラス (エラー発生時にのみ使用される).

エラー発生時にネイティブコードのスタックフレームをデコードするためのクラス
(スタックトレースの出力や FlatProfiler によるプロファイル取得に使用されている模様)

(また関連する定数値もこのクラス内に定義されている).


```
    ((cite: hotspot/src/share/vm/utilities/decoder.hpp))
    class Decoder: public StackObj {
```

### 使われ方(Usage)
以下のような箇所で使われている.

```
* VMError による処理 (print_stack_trace)

  VMError::print_stack_trace()
  -> frame::print_on_error()
     -> print_C_frame()
        -> Decoder::can_decode_C_frame_in_vm()
        -> os::dll_address_to_function_name()
           -> Decoder::demangle()  (<= Linux/Solaris 版の場合. Windows 版では呼ばれていない)
           -> Decoder::decode()

* VMError による処理 (report_and_die)

  VMError::report_and_die()
  -> VMError::report()
     -> frame::print_on_error()
        -> 同上

* FlatProfilerTask による処理

  FlatProfilerTask::task()
  -> FlatProfiler::record_vm_tick()
     -> os::dll_address_to_function_name()
        -> 同上
```

また, 関連する定数値が ElfFile クラスや ElfSymbolTable クラス, ElfStringTable クラス内で使用されている (See: ElfFile, ElfSymbolTable, ElfStringTable).




### 詳細(Details)
See: [here](../doxygen/classDecoder.html) for details

---
