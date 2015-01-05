---
layout: default
title: Interpreter に関する処理 ： 初期化処理 (1) ： Bytecode table の初期化処理
---
[Up](no3059kZk.html) [Top](../index.html)

#### Interpreter に関する処理 ： 初期化処理 (1) ： Bytecode table の初期化処理

--- 
## 概要(Summary)
Bytecode table の初期化処理は, HotSpot の起動時に呼び出される bytecodes_init() の中で行われる.

## 備考(Notes)
Bytecode table については Bytecodes クラスも参照 (See: Bytecodes).

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
(HotSpot の起動時処理) (See: <a href="no2114J7x.html">here</a> for details)
-&gt; Threads::create_vm()
   -&gt; init_globals()
      -&gt; bytecodes_init()
         -&gt; Bytecodes::initialize()
            -&gt; Bytecodes::def()
               -&gt; Bytecodes::compute_flags()
            -&gt; Bytecodes::pd_initialize()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### bytecodes_init()
See: [here](no3059KTA.html) for details
### Bytecodes::initialize()
See: [here](no3059XdG.html) for details
### Bytecodes::def(Code code, const char* name, const char* format, const char* wide_format, BasicType result_type, int depth, bool can_trap)
See: [here](no3059knM.html) for details
### Bytecodes::def(Code code, const char* name, const char* format, const char* wide_format, BasicType result_type, int depth, bool can_trap, Code java_code)
See: [here](no3059xxS.html) for details
### Bytecodes::compute_flags()
(#Under Construction)
See: [here](no3059-7Y.html) for details
### Bytecodes::pd_initialize() (sparc の場合)
See: [here](no3059LGf.html) for details
### Bytecodes::pd_initialize() (x86 の場合)
See: [here](no3059YQl.html) for details






