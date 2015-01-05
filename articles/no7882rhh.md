---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： Template Interpreter の処理中での Safepoint 停止処理  
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： Template Interpreter の処理中での Safepoint 停止処理  

--- 
## 概要(Summary)
Template Interpreter の場合, dispatchTable の中身を変更することで, 強制的な状態遷移を実現している.

より具体的に言うと, Safepoint が開始されると全ての codelet が Safepoint 処理を行うコードに置き換わる.
このため任意のバイトコードの先頭が Safepoint になる
(= インタープリタが次のバイトコードへ遷移する際に停止).

なお, 実際の停止処理は InterpreterRuntime の関数 (InterpreterRuntime::at_safepoint()) を呼ぶことで行い,
その際の IRT_ENTRY/IRT_END のチェックで停止させている
(See: [here](no7882haw.html) for details).

## 備考(Notes)
Template Interpreter は以下の 3つの dispatch table を持つ.
実際に使用するのは _active_table.
これに _normal_table の内容か _safept_table の内容がコピーされる.

* 通常時は Safepoint チェックを行わない dispatch table (= _normal_table) がコピーされている.

* Safepoint 停止が必要になると _safept_table の内容へと書き換えられる.
  Safepoint 処理が終了すると, また _normal_table の内容へと戻される.

(なお _active_table の変更処理は, SafepointSynchronize::begin() と SafepointSynchronize::end() の中で行われている
 (See: TemplateInterpreter::notice_safepoints(), TemplateInterpreter::ignore_safepoints())).


```cpp
    ((cite: hotspot/src/share/vm/interpreter/templateInterpreter.hpp))
      static DispatchTable _active_table;                           // the active    dispatch table (used by the interpreter for dispatch)
      static DispatchTable _normal_table;                           // the normal    dispatch table (used to set the active table in normal mode)
      static DispatchTable _safept_table;                           // the safepoint dispatch table (used to set the active table for safepoints)
```

_safept_table の中身は TemplateInterpreterGenerator::set_safepoints_for_all_bytes() で生成される.
テーブル内のエントリは, (bytecode や tos 状態に関わらず) 全て TemplateInterpreter::_safept_entry に格納されているコードを指している.
(See: TemplateInterpreterGenerator::set_safepoints_for_all_bytes(),
      TemplateInterpreterGenerator::generate_safept_entry_for())


## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
_safept_table の各エントリ
-&gt; TemplateInterpreter::_safept_entry に格納されているコード
   -&gt; TemplateInterpreterGenerator::generate_safept_entry_for() が生成したコード
      -&gt; InterpreterRuntime::at_safepoint()
         -&gt; IRT_ENTRY マクロ
            -&gt; (See: <a href="no7882haw.html">here</a> for details)
         -&gt; IRT_END マクロ
            -&gt; (See: <a href="no7882haw.html">here</a> for details)
               (Safepoint 処理が開始されていれば, この中で停止する)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateInterpreterGenerator::generate_safept_entry_for()  (sparc の場合)
See: [here](no3059A3y.html) for details
### TemplateInterpreterGenerator::generate_safept_entry_for()  (x86_64 の場合)
See: [here](no3059yAC.html) for details
### InterpreterRuntime::at_safepoint()
See: [here](no7882eXb.html) for details






