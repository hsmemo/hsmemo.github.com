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


```cpp
    ((cite: hotspot/src/share/vm/utilities/decoder.hpp))
    class Decoder: public StackObj {
```

### 使われ方(Usage)
以下のような箇所で使われている.

<div class="flow-abst"><pre>
* VMError による処理 (print_stack_trace)

  VMError::print_stack_trace()
  -&gt; frame::print_on_error()
     -&gt; print_C_frame()
        -&gt; Decoder::can_decode_C_frame_in_vm()
        -&gt; os::dll_address_to_function_name()
           -&gt; Decoder::demangle()  (&lt;= Linux/Solaris 版の場合. Windows 版では呼ばれていない)
           -&gt; Decoder::decode()

* VMError による処理 (report_and_die)

  VMError::report_and_die()
  -&gt; VMError::report()
     -&gt; frame::print_on_error()
        -&gt; 同上

* FlatProfilerTask による処理

  FlatProfilerTask::task()
  -&gt; FlatProfiler::record_vm_tick()
     -&gt; os::dll_address_to_function_name()
        -&gt; 同上
</pre></div>

また, 関連する定数値が ElfFile クラスや ElfSymbolTable クラス, ElfStringTable クラス内で使用されている (See: ElfFile, ElfSymbolTable, ElfStringTable).




### 詳細(Details)
See: [here](../doxygen/classDecoder.html) for details

---
