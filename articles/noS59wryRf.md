---
layout: default
title: Template Interpreter によるバイトコードの実行処理 ： 制御の移行 (if*, tableswitch, lookupswitch, goto*, jsr*, ret)
---
[Up](noaqS079AL.html) [Top](../index.html)

#### Template Interpreter によるバイトコードの実行処理 ： 制御の移行 (if*, tableswitch, lookupswitch, goto*, jsr*, ret)

--- 
## 概要(Summary)
各バイトコードはそれぞれ以下のテンプレートで処理される.

<!-- Turn-ON: (turn-on-orgtbl), Turn-OFF: (orgtbl-mode -1) -->
<!-- BEGIN RECEIVE ORGTBL table10981Ky2 -->
| Bytecode | Template |
|---|---|
| ifeq | TemplateTable::if_0cmp() |
| ifne | TemplateTable::if_0cmp() |
| iflt | TemplateTable::if_0cmp() |
| ifge | TemplateTable::if_0cmp() |
| ifgt | TemplateTable::if_0cmp() |
| ifle | TemplateTable::if_0cmp() |
| if_icmpeq | TemplateTable::if_icmp() |
| if_icmpne | TemplateTable::if_icmp() |
| if_icmplt | TemplateTable::if_icmp() |
| if_icmpge | TemplateTable::if_icmp() |
| if_icmpgt | TemplateTable::if_icmp() |
| if_icmple | TemplateTable::if_icmp() |
| if_acmpeq | TemplateTable::if_acmp() |
| if_acmpne | TemplateTable::if_acmp() |
| goto | TemplateTable::_goto() |
| jsr | TemplateTable::jsr() |
| ret | TemplateTable::ret() |
| tableswitch | TemplateTable::tableswitch() |
| lookupswitch | TemplateTable::lookupswitch() |
| ifnull | TemplateTable::if_nullcmp() |
| ifnonnull | TemplateTable::if_nullcmp() |
| goto_w | TemplateTable::goto_w() |
| jsr_w | TemplateTable::jsr_w() |
| ret | TemplateTable::wide_ret() |
| fast_linearswitch | TemplateTable::fast_linearswitch() |
| fast_binaryswitch | TemplateTable::fast_binaryswitch() |
<!-- END RECEIVE ORGTBL table10981Ky2 -->

<!-- 
#+ORGTBL: SEND table10981Ky2 orgtbl-to-gfm :no-escape t
| Bytecode           | Template                           |
|--------------------+------------------------------------|
| ifeq              | TemplateTable::if_0cmp()           |
| ifne              | TemplateTable::if_0cmp()           |
| iflt              | TemplateTable::if_0cmp()           |
| ifge              | TemplateTable::if_0cmp()           |
| ifgt              | TemplateTable::if_0cmp()           |
| ifle              | TemplateTable::if_0cmp()           |
| if_icmpeq         | TemplateTable::if_icmp()           |
| if_icmpne         | TemplateTable::if_icmp()           |
| if_icmplt         | TemplateTable::if_icmp()           |
| if_icmpge         | TemplateTable::if_icmp()           |
| if_icmpgt         | TemplateTable::if_icmp()           |
| if_icmple         | TemplateTable::if_icmp()           |
| if_acmpeq         | TemplateTable::if_acmp()           |
| if_acmpne         | TemplateTable::if_acmp()           |
| goto              | TemplateTable::_goto()             |
| jsr               | TemplateTable::jsr()               |
| ret               | TemplateTable::ret()               |
| tableswitch       | TemplateTable::tableswitch()       |
| lookupswitch      | TemplateTable::lookupswitch()      |
| ifnull            | TemplateTable::if_nullcmp()        |
| ifnonnull         | TemplateTable::if_nullcmp()        |
| goto_w            | TemplateTable::goto_w()            |
| jsr_w             | TemplateTable::jsr_w()             |
| ret               | TemplateTable::wide_ret()          |
| fast_linearswitch | TemplateTable::fast_linearswitch() |
| fast_binaryswitch | TemplateTable::fast_binaryswitch() |
-->


```cpp
    ((cite: hotspot/src/share/vm/interpreter/templateTable.cpp))
      //                                    interpr. templates
      // Java spec bytecodes                ubcp|disp|clvm|iswd  in    out   generator             argument
    ...
      def(Bytecodes::_ifeq                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , equal        );
      def(Bytecodes::_ifne                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , not_equal    );
      def(Bytecodes::_iflt                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , less         );
      def(Bytecodes::_ifge                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , greater_equal);
      def(Bytecodes::_ifgt                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , greater      );
      def(Bytecodes::_ifle                , ubcp|____|clvm|____, itos, vtos, if_0cmp             , less_equal   );
      def(Bytecodes::_if_icmpeq           , ubcp|____|clvm|____, itos, vtos, if_icmp             , equal        );
      def(Bytecodes::_if_icmpne           , ubcp|____|clvm|____, itos, vtos, if_icmp             , not_equal    );
      def(Bytecodes::_if_icmplt           , ubcp|____|clvm|____, itos, vtos, if_icmp             , less         );
      def(Bytecodes::_if_icmpge           , ubcp|____|clvm|____, itos, vtos, if_icmp             , greater_equal);
      def(Bytecodes::_if_icmpgt           , ubcp|____|clvm|____, itos, vtos, if_icmp             , greater      );
      def(Bytecodes::_if_icmple           , ubcp|____|clvm|____, itos, vtos, if_icmp             , less_equal   );
      def(Bytecodes::_if_acmpeq           , ubcp|____|clvm|____, atos, vtos, if_acmp             , equal        );
      def(Bytecodes::_if_acmpne           , ubcp|____|clvm|____, atos, vtos, if_acmp             , not_equal    );
      def(Bytecodes::_goto                , ubcp|disp|clvm|____, vtos, vtos, _goto               ,  _           );
      def(Bytecodes::_jsr                 , ubcp|disp|____|____, vtos, vtos, jsr                 ,  _           ); // result is not an oop, so do not transition to atos
      def(Bytecodes::_ret                 , ubcp|disp|____|____, vtos, vtos, ret                 ,  _           );
      def(Bytecodes::_tableswitch         , ubcp|disp|____|____, itos, vtos, tableswitch         ,  _           );
      def(Bytecodes::_lookupswitch        , ubcp|disp|____|____, itos, itos, lookupswitch        ,  _           );
    ...
      def(Bytecodes::_ifnull              , ubcp|____|clvm|____, atos, vtos, if_nullcmp          , equal        );
      def(Bytecodes::_ifnonnull           , ubcp|____|clvm|____, atos, vtos, if_nullcmp          , not_equal    );
      def(Bytecodes::_goto_w              , ubcp|____|clvm|____, vtos, vtos, goto_w              ,  _           );
      def(Bytecodes::_jsr_w               , ubcp|____|____|____, vtos, vtos, jsr_w               ,  _           );
    ...
      def(Bytecodes::_ret                 , ubcp|disp|____|iswd, vtos, vtos, wide_ret            ,  _           );
    ...
      def(Bytecodes::_fast_linearswitch   , ubcp|disp|____|____, itos, vtos, fast_linearswitch   ,  _           );
      def(Bytecodes::_fast_binaryswitch   , ubcp|disp|____|____, itos, vtos, fast_binaryswitch   ,  _           );
    ...
```

## 備考(Notes)
* fast_linearswitch や fast_binaryswitch は rewrite 処理により生成されるバイトコード (See: [here](no3059AfB.html) for details)

* 一部の命令については,
  逆向きの分岐(backward branch)の実行数が一定回数を超えると JIT コンパイルが実行されて高速化される
  (See: [here](no-umZIXCH.html) for details)

## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
<div class="flow-abst"><pre>
TemplateTable::if_0cmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; * jsr 用のコードを生成する場合:
           -&gt; (1) Lbcp の値を変更する
              (2) 次のバイトコードに対応するテンプレートへとジャンプする
                  -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード
         * jsr 以外用のコードを生成する場合:
           -&gt; (1) backedge_counter のカウンタ値をインクリメントする
                  -&gt; InterpreterMacroAssembler::increment_backedge_counter() が生成したコード
              (1) methodDataOop に関するチェック (&amp; 必要ならば methodDataOop の生成) を行う
                  -&gt; InterpreterMacroAssembler::test_invocation_counter_for_mdp() が生成したコード
              (1) カウンタ値と閾値を比較し, 閾値を超えていれば JIT コンパイルを行う.
                  あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する.
                  (遷移した場合は以降の処理は実行されない)
                  -&gt; InterpreterMacroAssembler::test_backedge_count_for_osr() が生成したコード
                     -&gt; (See: <a href="no-umZIXCH.html">here</a> for details)
              (1) Lbcp の値を変更する
              (1) 次のバイトコードに対応するテンプレートへとジャンプする
                  -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード
   
TemplateTable::if_icmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::if_acmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::_goto() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::jsr() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::ret() が生成したコード
-&gt; (1) Lbcp の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::tableswitch() が生成したコード
-&gt; (1) Lbcp の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::lookupswitch() が生成したコード
-&gt; InterpreterMacroAssembler::stop() が生成したコード
   
TemplateTable::if_nullcmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::goto_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::jsr_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::wide_ret() が生成したコード
-&gt; (1) Lbcp の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::fast_linearswitch() が生成したコード
-&gt; (1) Lbcp の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::fast_binaryswitch() が生成したコード
-&gt; (1) Lbcp の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード
</pre></div>


### x86_64 の場合
<div class="flow-abst"><pre>
TemplateTable::if_0cmp() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; * jsr 用のコードを生成する場合:
        -&gt; (1) r13 の値を変更する
           (2) 次のバイトコードに対応するテンプレートへとジャンプする
               -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード
      * jsr 以外用のコードを生成する場合:
        -&gt; (1) r13 の値を変更する
           (1) backedge_counter のカウンタ値をインクリメントする
               -&gt; InterpreterMacroAssembler::increment_mask_and_jump() が生成したコード
           (1) methodDataOop に関するチェック (&amp; 必要ならば methodDataOop の生成) を行う
               -&gt; InterpreterMacroAssembler::test_method_data_pointer() が生成したコード
               -&gt; InterpreterRuntime::profile_method()
           (1) カウンタ値と閾値を比較し, 閾値を超えていれば JIT コンパイルを行う.
               あるいは, 既に JIT コンパイル済みのコードがあれば, OSR を行ってそのコードに遷移する.
               (遷移した場合は以降の処理は実行されない)
               -&gt; InterpreterRuntime::frequency_counter_overflow()
                  -&gt; (See: <a href="no-umZIXCH.html">here</a> for details)
               -&gt; SharedRuntime::OSR_migration_begin()
                  -&gt; (See: <a href="norlT6uoVb.html">here</a> for details)
           (1) 次のバイトコードに対応するテンプレートへとジャンプする
               -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード
   
TemplateTable::if_icmp() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::if_acmp() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::_goto() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::jsr() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::ret() が生成したコード
-&gt; (1) r13 の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::tableswitch() が生成したコード
-&gt; (1) r13 の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード

TemplateTable::lookupswitch() が生成したコード
-&gt; InterpreterMacroAssembler::stop() が生成したコード
   
TemplateTable::if_nullcmp() が生成したコード
-&gt; InterpreterMacroAssembler::if_cmp() が生成したコード
   -&gt; TemplateTable::branch() が生成したコード
      -&gt; (同上)

TemplateTable::goto_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::jsr_w() が生成したコード
-&gt; TemplateTable::branch() が生成したコード
   -&gt; (同上)

TemplateTable::wide_ret() が生成したコード
-&gt; (1) r13 の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_next() が生成したコード

TemplateTable::fast_linearswitch() が生成したコード
-&gt; (1) r13 の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード

TemplateTable::fast_binaryswitch() が生成したコード
-&gt; (1) r13 の値を変更する
   (2) 次のバイトコードに対応するテンプレートへとジャンプする
       -&gt; InterpreterMacroAssembler::dispatch_only() が生成したコード
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::if_0cmp() (sparc の場合)
See: [here](no10981XKl.html) for details
### InterpreterMacroAssembler::if_cmp() (sparc の場合)
See: [here](no10981RZM.html) for details
### TemplateTable::if_icmp() (sparc の場合)
See: [here](no1098198M.html) for details
### TemplateTable::if_acmp() (sparc の場合)
See: [here](no10981MCt.html) for details
### TemplateTable::_goto() (sparc の場合)
See: [here](no10981AAi.html) for details
### TemplateTable::jsr() (sparc の場合)
See: [here](no10981NYQ.html) for details
### TemplateTable::ret() (sparc の場合)
See: [here](no10981lqO.html) for details
### TemplateTable::tableswitch() (sparc の場合)
See: [here](no10981bV1.html) for details
### TemplateTable::lookupswitch() (sparc の場合)
See: [here](no10981nzQ.html) for details
### TemplateTable::if_nullcmp() (sparc の場合)
See: [here](no10981ZMz.html) for details
### TemplateTable::goto_w() (sparc の場合)
See: [here](no10981ne0.html) for details
### TemplateTable::jsr_w() (sparc の場合)
See: [here](no10981NRc.html) for details
### TemplateTable::wide_ret() (sparc の場合)
See: [here](no10981mkh.html) for details
### TemplateTable::fast_linearswitch() (sparc の場合)
See: [here](no10981OSj.html) for details
### TemplateTable::fast_binaryswitch() (sparc の場合)
See: [here](no10981OZX.html) for details
### TemplateTable::branch() (sparc の場合)
See: [here](no10981cyM.html) for details
### InterpreterMacroAssembler::increment_backedge_counter() (sparc の場合)
See: [here](no109817Nm.html) for details
### InterpreterMacroAssembler::test_invocation_counter_for_mdp() (sparc の場合)
See: [here](no10981v9y.html) for details
### InterpreterMacroAssembler::test_backedge_count_for_osr() (sparc の場合)
See: [here](no10981SYp.html) for details

### TemplateTable::if_0cmp() (x86_64 の場合)          
See: [here](no1098167W.html) for details
### TemplateTable::if_icmp() (x86_64 の場合)          
See: [here](no1098171p.html) for details
### TemplateTable::if_acmp() (x86_64 の場合)          
See: [here](no10981VK2.html) for details
### TemplateTable::_goto() (x86_64 の場合)
See: [here](no10981AAi.html) for details
### TemplateTable::jsr() (x86_64 の場合)
See: [here](no10981NYQ.html) for details
### TemplateTable::ret (x86_64 の場合)
See: [here](no10981HUF.html) for details
### TemplateTable::tableswitch() (x86_64 の場合)      
See: [here](no10981hoR.html) for details
### TemplateTable::lookupswitch() (x86_64 の場合)     
See: [here](no10981ibw.html) for details
### TemplateTable::if_nullcmp() (x86_64 の場合)       
See: [here](no10981IAw.html) for details
### TemplateTable::goto_w() (x86_64 の場合)
See: [here](no10981ne0.html) for details
### TemplateTable::jsr_w() (x86_64 の場合)
See: [here](no10981NRc.html) for details
### TemplateTable::wide_ret() (x86_64 の場合)
See: [here](no10981UeL.html) for details
### TemplateTable::fast_linearswitch() (x86_64 の場合)
See: [here](no10981u5L.html) for details
### TemplateTable::fast_binaryswitch() (x86_64 の場合)
See: [here](no10981vsq.html) for details
### TemplateTable::branch() (x86_64 の場合)
See: [here](no10981VfS.html) for details






