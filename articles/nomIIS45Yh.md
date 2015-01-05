---
layout: default
title: Field に関する処理 ： フィールドアクセス処理 ： Template Interpreter の場合
---
[Up](noEVXMRz5w.html) [Top](../index.html)

#### Field に関する処理 ： フィールドアクセス処理 ： Template Interpreter の場合

--- 
## 概要(Summary)
(#Under Construction)



volatile フィールドの場合には, 以下の箇所でメモリバリアが必要になる.
ただし, x86 では StoreLoad しか必要ないため "3." の箇所でのみバリアが張られている.
SPARC でも実行環境が V9 以降なら同様の箇所にしか張られない.

  1. TemplateTable::getfield_or_static() の最後に LoadLoad  & LoadStore  バリア
  2. TemplateTable::putfield_or_static() の最初に LoadStore & StoreStore バリア
  3. TemplateTable::putfield_or_static() の最後に StoreLoad バリア     (<= x86 版では StoreStore も指定されているが必要ないような...?? #TODO)


## 備考(Notes)
* まだ CPCache のエントリが作られていない場合は, 
  TemplateTable::resolve_cache_and_index() が生成するコード内でエントリを作る処理が行われる.

* getfield 命令および putfield 命令の場合は, 
  初回の実行時にそれぞれ Bytecodes::_fast_?getfield や Bytecodes::_fast_?putfield への書き換えを行う 
  (See: [here](no7882vBO.html) for details).
  そのため, 二回目以降の実行は少し速くなる.
  
  なお, getstatic 命令や putstatic 命令の場合には書き換えは行われない.


## 処理の流れ (概要)(Execution Flows : Summary)
### sparc の場合
#### getfield 命令の処理
<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::getfield() が生成するコード
  -&gt; TemplateTable::getfield_or_static() が生成するコード
     -&gt; (1) CPCache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (1) 既に作成済みであれば, それを取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う
                      -&gt; InterpreterRuntime::resolve_get_put()
                         -&gt; (See: <a href="no7882NqI.html">here</a> for details)
                  (2) まだ作成されて無かった場合は, ここで取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) CPCache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
        (1) ロード処理 (型情報に応じて適切なロードを行う)
   
        (1) getfield 命令の場合 (= getstatic 命令ではない場合) には, 高速版に書き換える
            -&gt; TemplateTable::patch_bytecode() が生成するコード
   
        (1) volatile field だった場合はメモリバリア処理を行う
            -&gt; TemplateTable::volatile_barrier() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -&gt; (1) CPCache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) CPCache entry の中身を取り出す

     (1) ロード処理 (型情報に応じて適切なロードを行う)
  
     (1) volatile field だった場合はメモリバリア処理を行う
         -&gt; TemplateTable::volatile_barrier() が生成するコード
</pre></div>

#### getstatic 命令の処理
<div class="flow-abst"><pre>
TemplateTable::getstatic() が生成するコード
-&gt; TemplateTable::getfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

#### putfield 命令の処理
<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::putfield() が生成するコード
  -&gt; TemplateTable::putfield_or_static() が生成するコード
     -&gt; (1) CPCache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (同上)
   
        (1) CPCache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
        (1) volatile field だった場合は (必要に応じて) メモリバリア処理を行う
            -&gt; TemplateTable::volatile_barrier() が生成するコード
   
        (1) ストア処理 (型情報に応じて適切なストアを行う)
   
        (1) putfield 命令の場合 (= putstatic 命令ではない場合) には, 高速版に書き換える
            -&gt; TemplateTable::patch_bytecode() が生成するコード

        (1) volatile field だった場合はメモリバリア処理を行う
            -&gt; TemplateTable::volatile_barrier() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -&gt; (1) CPCache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) volatile field だった場合は (必要に応じて) メモリバリア処理を行う
         -&gt; TemplateTable::volatile_barrier() が生成するコード

     (1) CPCache entry の中身を取り出す

     (1) ストア処理 (型情報に応じて適切なストアを行う)
  
     (1) volatile field だった場合はメモリバリア処理を行う
         -&gt; TemplateTable::volatile_barrier() が生成するコード
</pre></div>

#### putstatic 命令の処理
<div class="flow-abst"><pre>
TemplateTable::putstatic() が生成するコード
-&gt; TemplateTable::putfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>


### x86_64 の場合
#### getfield 命令の処理
<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::getfield() が生成するコード
  -&gt; TemplateTable::getfield_or_static() が生成するコード
     -&gt; (1) CPCache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (1) 既に作成済みであれば, それを取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う
                      -&gt; InterpreterRuntime::resolve_get_put()
                         -&gt; (See: <a href="no7882NqI.html">here</a> for details)
                  (2) まだ作成されて無かった場合は, ここで取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) CPCache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
        (1) ロード処理 (型情報に応じて適切なロードを行う)
   
        (1) getfield 命令の場合 (= getstatic 命令ではない場合) には, 高速版に書き換える
            -&gt; TemplateTable::patch_bytecode() が生成するコード
   
        (1) (x86 ではメモリバリアは不要なので張っていない)

* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -&gt; (1) CPCache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) CPCache entry の中身を取り出す

     (1) ロード処理 (型情報に応じて適切なロードを行う)
  
     (1) (x86 ではメモリバリアは不要なので張っていない)
</pre></div>

#### getstatic 命令の処理
<div class="flow-abst"><pre>
TemplateTable::getstatic() が生成するコード
-&gt; TemplateTable::getfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

#### putfield 命令の処理
<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::putfield() が生成するコード
  -&gt; TemplateTable::putfield_or_static() が生成するコード
     -&gt; (1) CPCache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (同上)
   
        (1) CPCache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
        (1) (x86 ではメモリバリアは不要なので張っていない)
   
        (1) ストア処理 (型情報に応じて適切なストアを行う)
   
        (1) putfield 命令の場合 (= putstatic 命令ではない場合) には, 高速版に書き換える
            -&gt; TemplateTable::patch_bytecode() が生成するコード

        (1) volatile field だった場合はメモリバリア処理を行う
            -&gt; TemplateTable::volatile_barrier() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -&gt; (1) CPCache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) (x86 ではメモリバリアは不要なので張っていない)

     (1) CPCache entry の中身を取り出す

     (1) ストア処理 (型情報に応じて適切なストアを行う)
  
     (1) volatile field だった場合はメモリバリア処理を行う
         -&gt; TemplateTable::volatile_barrier() が生成するコード
</pre></div>

#### putstatic 命令の処理
<div class="flow-abst"><pre>
TemplateTable::putstatic() が生成するコード
-&gt; TemplateTable::putfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateTable::getfield() (sparc 版)
See: [here](no7882jjy.html) for details
### TemplateTable::getstatic() (sparc 版)
See: [here](no7882VtB.html) for details
### TemplateTable::getfield_or_static() (sparc 版)
See: [here](no7882i3H.html) for details
### TemplateTable::resolve_cache_and_index() (sparc 版)
(#Under Construction)
See: [here](no78828LU.html) for details
### InterpreterMacroAssembler::get_cache_and_index_at_bcp() (sparc 版)
See: [here](no78828SI.html) for details
### InterpreterMacroAssembler::get_cache_index_at_bcp() (sparc 版)
See: [here](no7882JdO.html) for details
### TemplateTable::load_field_cp_cache_entry() (sparc 版)
See: [here](no7882JWa.html) for details
### TemplateTable::fast_accessfield() (sparc 版)
See: [here](no7882Wgg.html) for details
### TemplateTable::volatile_barrier() (sparc 版)
See: [here](no7882j_C.html) for details
### MacroAssembler::membar() (sparc 版)
See: [here](no78829TP.html) for details

### TemplateTable::putfield() (sparc 版)
See: [here](no7882jqm.html) for details
### TemplateTable::putstatic() (sparc 版)
See: [here](no7882w0s.html) for details
### TemplateTable::putfield_or_static() (sparc 版)
See: [here](no78829-y.html) for details
### TemplateTable::fast_storefield() (sparc 版)
See: [here](no7882vIC.html) for details

### TemplateTable::getfield() (x86_64 版)
See: [here](no7882WnU.html) for details
### TemplateTable::getstatic() (x86_64 版)
See: [here](no7882jxa.html) for details
### TemplateTable::getfield_or_static() (x86_64 版)
See: [here](no7882w7g.html) for details
### TemplateTable::fast_accessfield() (x86_64 版)
See: [here](no78829Fn.html) for details
### TemplateTable::resolve_cache_and_index() (x86_64 版)
(#Under Construction)
See: [here](no7882Xob.html) for details
### InterpreterMacroAssembler::get_cache_and_index_at_bcp() (x86_64 版)
See: [here](no7882kyh.html) for details
### InterpreterMacroAssembler::get_cache_index_at_bcp() (x86_64 版)
See: [here](no7882x8n.html) for details
### TemplateTable::load_field_cp_cache_entry() (x86_64 版)
See: [here](no7882-Gu.html) for details
### TemplateTable::volatile_barrier() (x86_64 版)
See: [here](no7882wJJ.html) for details
### MacroAssembler::membar() (x86_64 版)
See: [here](no7882KeV.html) for details

### TemplateTable::putfield() (x86_64 版)
See: [here](no7882KQt.html) for details
### TemplateTable::putstatic() (x86_64 版)
See: [here](no7882Xaz.html) for details
### TemplateTable::putfield_or_static() (x86_64 版)
See: [here](no7882JkC.html) for details
### TemplateTable::fast_storefield() (x86_64 版)
See: [here](no7882WuI.html) for details






