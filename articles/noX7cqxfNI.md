---
layout: default
title: 実行時コンスタントプールのシンボルの解決処理(resolution) (1) ： 解決処理の開始契機 ： Interpreter の場合
---
[Up](noDbg01tAk.html) [Top](../index.html)

#### 実行時コンスタントプールのシンボルの解決処理(resolution) (1) ： 解決処理の開始契機 ： Interpreter の場合

--- 
## 概要(Summary)
インタープリタは, 以下のバイトコード命令を実行した際に解決処理を実行する
(詳細は Java 仮想マシン仕様 5.4.3 "Resolution" 参照).

  * new, anewarray, multianewarray
  * getfield, getstatic, putfield, putstatic
  * invokeinterface, invokespecial, invokestatic, invokevirtual, invokedynamic
  * checkcast, instanceof
  * ldc, ldc_w
  * fast_aldc, fast_aldc_w  (※ fast_aldc 及び fast_aldc_w は rewrite 処理で導入される HotSpot 独自バイトコード (See: [here](no3059AfB.html) for details))

インタープリタ種別(Template Interpreter, C++ Interpreter)によって処理の流れは多少異なる
(また Template Interpreter の場合はアーキテクチャ(sparc, x86)によっても若干異なる).
ただし, どの場合も最終的にはランタイム(InterpreterRuntime)での処理に到達し, 
そこから constantPoolOopDesc や LinkResolver が呼び出されることで解決処理が行われる.

なおインタープリタでは, フィールドやメソッドの解決を高速化するため, 
constantPoolCacheOop というキャッシュを用いる.
このため, これらの解決時にはインタープリタは constantPoolCacheOop 経由で解決結果を取得する
(constantPoolCacheOop は, キャッシュされていなければランタイムを呼び出して解決処理を行う.
 解決結果は constantPoolCacheOop 内に登録されるため,
 フィールドやメソッドの解決処理は各参照に付き最初の1回のみ実行される).


## 処理の流れ (概要)(Execution Flows : Summary)
### Template Interpreter の場合
#### sparc の場合
* new 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::_new() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::_new() で処理を行う
       -&gt; InterpreterRuntime::_new()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* anewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::anewarray() が生成するコード
-&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得.
       また現在実行中のメソッドに対応した constant pool を取得.
       -&gt; InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
       -&gt; InterpreterMacroAssembler::get_constant_pool() が生成するコード
   (1) InterpreterRuntime::anewarray() で処理を行う
       -&gt; InterpreterRuntime::anewarray()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* multianewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::multianewarray() が生成するコード
-&gt; InterpreterRuntime::multianewarray()
   -&gt; (下記参照)
</pre></div>

* getfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::getfield() が生成するコード
  -&gt; TemplateTable::getfield_or_static() が生成するコード
     -&gt; (1) constant pool cache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (1) 既に作成済みであれば, それを取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う (この場合はフィールドアクセスなので, InterpreterRuntime::resolve_get_put() を呼び出す)
                      -&gt; InterpreterRuntime::resolve_get_put()
                         -&gt; ランタイムで処理を行う(下記参照)
                  (3) まだ作成されて無かった場合は, ここで取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) constant pool cache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -&gt; (1) constant pool cache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
</pre></div>

* getstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
TemplateTable::getstatic() が生成するコード
-&gt; TemplateTable::getfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

* putfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::putfield() が生成するコード
  -&gt; TemplateTable::putfield_or_static() が生成するコード
     -&gt; (1) constant pool cache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (同上)
   
        (1) constant pool cache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード
   
* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -&gt; (1) constant pool cache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
</pre></div>

* putstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
TemplateTable::putstatic() が生成するコード
-&gt; TemplateTable::putfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

* invokevirtual 命令の処理  (See: [here](no3059-tw.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokevirtual() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
             -&gt; (1) 既に作成済みであれば, それを取得
                    -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic 以外の invoke* なので, InterpreterRuntime::resolve_invoke() を呼び出す)
                    -&gt; InterpreterRuntime::resolve_invoke()
                       -&gt; ランタイムで処理を行う(下記参照)
                (3) まだ作成されて無かった場合は, ここで取得
                    -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>

* invokeinterface 命令の処理  (See: [here](no3059L42.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokeinterface() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -&gt; (同上)
</pre></div>

* invokestatic 命令の処理  (See: [here](nozMknLl1S.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokestatic() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -&gt; (同上)
</pre></div>

* invokespecial 命令の処理  (See: [here](noZ4XAqR7E.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokespecial() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -&gt; (同上)
</pre></div>

* invokedynamic 命令の処理 (See: [here](noikurSEOv.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokedynamic() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
             -&gt; (1) 既に作成済みであれば, それを取得
                    -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic なので, InterpreterRuntime::resolve_invokedynamic() を呼び出す)
                    -&gt; InterpreterRuntime::resolve_invokedynamic()
                       -&gt; ランタイムで処理を行う(下記参照)
                (3) まだ作成されて無かった場合は, ここで取得
                    -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>

* checkcast 命令の処理 (See: [here](no5F7e42op.html) for details)

<div class="flow-abst"><pre>
TemplateTable::checkcast() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -&gt; InterpreterRuntime::quicken_io_cc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* instanceof 命令の処理 (See: [here](no5F7e42op.html) for details)

<div class="flow-abst"><pre>
TemplateTable::instanceof() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -&gt; InterpreterRuntime::quicken_io_cc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* ldc, ldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

<div class="flow-abst"><pre>
TemplateTable::ldc() が生成するコード
-&gt; (1) constant pool をチェックして, ロード対象が未解決の文字列もしくは未解決のクラスかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 1byte/2byte を取得する
              -&gt; ldub() が生成するコード (ldc の場合)
              -&gt; InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード (ldc_w の場合)
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 対象が解決済みかどうかを調べる

   (1) 対象が未解決の文字列／クラスであれば InterpreterRuntime::ldc() で処理を行う
       -&gt; InterpreterRuntime::ldc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* fast_aldc, fast_aldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

<div class="flow-abst"><pre>
TemplateTable::fast_aldc() が生成するコード
-&gt; TemplateTable::resolve_cache_and_index() が生成するコード
   -&gt; (1) 既に作成済みであれば, それを取得
          -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
      (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は fast_aldc なので, InterpreterRuntime::resolve_ldc() を呼び出す)
          -&gt; InterpreterRuntime::resolve_ldc()
             -&gt; ランタイムで処理を行う(下記参照)
      (3) まだ作成されて無かった場合は, ここで取得
          -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>

#### x86-64 の場合
* new 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::_new() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::_new() で処理を行う
       -&gt; InterpreterRuntime::_new()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* anewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::anewarray() が生成するコード
-&gt; (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得.
       また現在実行中のメソッドに対応した constant pool を取得.
       -&gt; InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
       -&gt; InterpreterMacroAssembler::get_constant_pool() が生成するコード
   (1) InterpreterRuntime::anewarray() で処理を行う
       -&gt; InterpreterRuntime::anewarray()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* multianewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

<div class="flow-abst"><pre>
TemplateTable::multianewarray() が生成するコード
-&gt; InterpreterRuntime::multianewarray()
   -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* getfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::getfield() が生成するコード
  -&gt; TemplateTable::getfield_or_static() が生成するコード
     -&gt; (1) constant pool cache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (1) 既に作成済みであれば, それを取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う (この場合はフィールドアクセスなので, InterpreterRuntime::resolve_get_put() を呼び出す)
                      -&gt; InterpreterRuntime::resolve_get_put()
                         -&gt; ランタイムで処理を行う(下記参照)
                  (3) まだ作成されて無かった場合は, ここで取得
                      -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) constant pool cache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -&gt; (1) constant pool cache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
</pre></div>

* getstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
TemplateTable::getstatic() が生成するコード
-&gt; TemplateTable::getfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

* putfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
* 1 回目
  TemplateTable::putfield() が生成するコード
  -&gt; TemplateTable::putfield_or_static() が生成するコード
     -&gt; (1) constant pool cache entry を取得
            -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
               -&gt; (同上)
   
        (1) constant pool cache entry の中身を取り出す
            -&gt; TemplateTable::load_field_cp_cache_entry() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -&gt; (1) constant pool cache entry を取得
         -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
</pre></div>

* putstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

<div class="flow-abst"><pre>
TemplateTable::putstatic() が生成するコード
-&gt; TemplateTable::putfield_or_static() が生成するコード
   -&gt; (同上)
</pre></div>

* invokevirtual 命令の処理  (See: [here](no3059-tw.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokevirtual() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::prepare_invoke() が生成するコード
          -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
             -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
                -&gt; (1) 既に作成済みであれば, それを取得
                       -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                   (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic 以外の invoke* なので, InterpreterRuntime::resolve_invoke() を呼び出す)
                       -&gt; InterpreterRuntime::resolve_invoke()
                          -&gt; ランタイムで処理を行う(下記参照)
                   (3) まだ作成されて無かった場合は, ここで取得
                       -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>

* invokeinterface 命令の処理  (See: [here](no3059L42.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokeinterface() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::prepare_invoke() が生成するコード
          -&gt; (同上)
</pre></div>

* invokestatic 命令の処理  (See: [here](nozMknLl1S.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokestatic() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::prepare_invoke() が生成するコード
          -&gt; (同上)
</pre></div>

* invokespecial 命令の処理  (See: [here](noZ4XAqR7E.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokespecial() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::prepare_invoke() が生成するコード
          -&gt; (同上)
</pre></div>

* invokedynamic 命令の処理 (See: [here](noikurSEOv.html) for details)

<div class="flow-abst"><pre>
TemplateTable::invokedynamic() が生成するコード
-&gt; (1) constant pool cache entry を取得し, 中身を取り出す
       -&gt; TemplateTable::prepare_invoke() が生成するコード
          -&gt; TemplateTable::load_invoke_cp_cache_entry() が生成するコード
             -&gt; TemplateTable::resolve_cache_and_index() が生成するコード
                -&gt; (1) 既に作成済みであれば, それを取得
                       -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                   (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic なので, InterpreterRuntime::resolve_invokedynamic() を呼び出す)
                       -&gt; InterpreterRuntime::resolve_invokedynamic()
                          -&gt; ランタイムで処理を行う(下記参照)
                   (3) まだ作成されて無かった場合は, ここで取得
                       -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>

* checkcast 命令の処理 (See: [here](no5F7e42op.html) for details)

<div class="flow-abst"><pre>
TemplateTable::checkcast() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -&gt; InterpreterRuntime::quicken_io_cc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* instanceof 命令の処理 (See: [here](no5F7e42op.html) for details)

<div class="flow-abst"><pre>
TemplateTable::instanceof() が生成するコード
-&gt; (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -&gt; (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -&gt; InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -&gt; InterpreterRuntime::quicken_io_cc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* ldc, ldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

<div class="flow-abst"><pre>
TemplateTable::ldc() が生成するコード
-&gt; (1) constant pool をチェックして, ロード対象が未解決の文字列もしくは未解決のクラスかどうかを確認する.
       -&gt; (1) 現在のバイトコード内から constant pool の index を示す 1byte/2byte を取得する
              -&gt; ldub() が生成するコード (ldc の場合)
              -&gt; InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード (ldc_w の場合)
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -&gt; InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 対象が解決済みかどうかを調べる

   (1) 対象が未解決の文字列／クラスであれば InterpreterRuntime::ldc() で処理を行う
       -&gt; InterpreterRuntime::ldc()
          -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

* fast_aldc, fast_aldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

<div class="flow-abst"><pre>
TemplateTable::fast_aldc() が生成するコード
-&gt; TemplateTable::resolve_cache_and_index() が生成するコード
   -&gt; (1) 既に作成済みであれば, それを取得
          -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
      (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は fast_aldc なので, InterpreterRuntime::resolve_ldc() を呼び出す)
          -&gt; InterpreterRuntime::resolve_ldc()
             -&gt; ランタイムで処理を行う(下記参照)
      (3) まだ作成されて無かった場合は, ここで取得
          -&gt; InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
</pre></div>


### C++ Interpreter の場合
<div class="flow-abst"><pre>
BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
-&gt; * new の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::_new()
              -&gt; ランタイムで処理を行う(下記参照)

   * anewarray の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::anewarray()
              -&gt; ランタイムで処理を行う(下記参照)

   * multianewarray の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::multianewarray()
              -&gt; ランタイムで処理を行う(下記参照)

   * getfield, putfield, getstatic, putstatic の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::resolve_get_put()
              -&gt; ランタイムで処理を行う(下記参照)

   * invokeinterface, invokevirtual, invokespecial, invokestatic の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::resolve_invoke()
              -&gt; ランタイムで処理を行う(下記参照)

   * invokedynamic の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::resolve_invokedynamic()
              -&gt; ランタイムで処理を行う(下記参照)

   * instanceof, checkcast の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::quicken_io_cc()
              -&gt; ランタイムで処理を行う(下記参照)

   * ldc, ldc_w の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::ldc()
              -&gt; ランタイムで処理を行う(下記参照)

   * fast_aldc, fast_aldc_w の場合
     -&gt; CALL_VM() マクロ
        -&gt; CALL_VM_NOCHECK() マクロ
           -&gt; InterpreterRuntime::resolve_ldc()
              -&gt; ランタイムで処理を行う(下記参照)
</pre></div>

### 両 Interpreter 共通の処理 (InterpreterRuntime の処理)
* new 命令の処理 (See: [here](no28916DqA.html) for details)

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::_new()
   -&gt; (1) 対象のクラスの解決処理を行う
          -&gt; constantPoolOopDesc::klass_at()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
</pre></div>

* anewarray 命令の処理 (See: [here](no28916DqA.html) for details)

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::anewarray()
   -&gt; (1) 対象のクラスの解決処理を行う
          -&gt; constantPoolOopDesc::klass_at()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
</pre></div>

* multianewarray 命令の処理 (See: [here](no28916DqA.html) for details)

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::multianewarray()
   -&gt; (1) 対象のクラスの解決処理を行う
          -&gt; constantPoolOopDesc::klass_at()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
</pre></div>

* getfield, getstatic, putfield, putstatic 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::resolve_get_put()
   -&gt; (1) 対象のフィールド名の解決を行う
          -&gt; LinkResolver::resolve_field()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)

      (1) 解決結果を constant pool cache entry にキャッシュしておく
          -&gt; ConstantPoolCacheEntry::set_field()
</pre></div>

* invokevirtual, invokeinterface, invokespecial, invokestatic 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::resolve_invoke()
   -&gt; (1) 対象のメソッド名の解決を行う
          -&gt; LinkResolver::resolve_invoke()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)

      (1) 解決結果を constant pool cache entry にキャッシュしておく
          -&gt; ConstantPoolCacheEntry::set_method()         (invokeinterface 以外の場合(※))
          -&gt; ConstantPoolCacheEntry::set_interface_call() (invokeinterface の場合(※))

(※) ただし, invokeinterface であっても 
     java.lang.Object のメソッドの場合には ConstantPoolCacheEntry::set_method() を使用する.
     (これは Java 仮想マシン仕様のコーナーケースとのこと #TODO)
</pre></div>

* invokedynamic 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::resolve_invokedynamic()
   -&gt; #TODO
</pre></div>

* checkcast, instanceof 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::quicken_io_cc()
   -&gt; (1) 対象のクラスの解決処理を行う
          -&gt; constantPoolOopDesc::klass_at()
             -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
</pre></div>

* ldc, ldc_w 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::ldc()
   -&gt; * ロード対象がクラスオブジェクトの場合:
        -&gt; constantPoolOopDesc::klass_at()
           -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
      * ロード対象が文字列オブジェクトの場合:
        -&gt; constantPoolOopDesc::string_at()
           -&gt; (See: <a href="no-VW6a46T.html">here</a> for details)
</pre></div>

* fast_aldc, fast_aldc_w 命令の処理

<div class="flow-abst"><pre>
各インタープリタの処理から呼び出される(上記参照)
-&gt; InterpreterRuntime::resolve_ldc()
   -&gt; Bytecode_loadconstant::resolve_constant()
      -&gt; 
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterMacroAssembler::get_cpool_and_tags() (sparc の場合)
See: [here](no19018mXV.html) for details
### InterpreterMacroAssembler::get_constant_pool() (sparc の場合)
See: [here](no19018qpZ.html) for details
### InterpreterMacroAssembler::get_cpool_and_tags() (x86_64 の場合)
See: [here](no19018oSv.html) for details
### InterpreterMacroAssembler::get_constant_pool() (x86_64 の場合)
See: [here](no19018CDM.html) for details
### InterpreterMacroAssembler::get_method() (x86_64 の場合)
See: [here](no19018CKA.html) for details

### TemplateTable::resolve_cache_and_index() (sparc 版)
(#Under Construction)
See: [here](no78828LU.html) for details
### InterpreterMacroAssembler::get_cache_and_index_at_bcp() (sparc 版)
See: [here](no78828SI.html) for details
### InterpreterMacroAssembler::get_cache_index_at_bcp() (sparc 版)
See: [here](no7882JdO.html) for details
### TemplateTable::load_field_cp_cache_entry() (sparc 版)
See: [here](no7882JWa.html) for details
### TemplateTable::resolve_cache_and_index() (x86_64 版)
(#Under Construction)
See: [here](no7882Xob.html) for details
### InterpreterMacroAssembler::get_cache_and_index_at_bcp() (x86_64 版)
See: [here](no7882kyh.html) for details
### InterpreterMacroAssembler::get_cache_index_at_bcp() (x86_64 版)
See: [here](no7882x8n.html) for details
### TemplateTable::load_field_cp_cache_entry() (x86_64 版)
See: [here](no7882-Gu.html) for details

### BytecodeInterpreter::run() (もしくは BytecodeInterpreter::runWithChecks())
(#Under Construction)


### InterpreterRuntime::_new()
See: [here](no344NwA.html) for details
### InterpreterRuntime::anewarray()
See: [here](no344btl.html) for details
### InterpreterRuntime::multianewarray()
See: [here](no38009YT.html) for details

### InterpreterRuntime::resolve_get_put()
(#Under Construction)
See: [here](no7882j4O.html) for details
### ConstantPoolCacheEntry::set_field()
See: [here](no7882x1z.html) for details

### InterpreterRuntime::resolve_invoke()
See: [here](no19018zXH.html) for details
### ConstantPoolCacheEntry::set_method()
(#Under Construction)
See: [here](no190185dx.html) for details
### ConstantPoolCacheEntry::set_interface_call()
(#Under Construction)
See: [here](no19018fQZ.html) for details

### InterpreterRuntime::quicken_io_cc()
See: [here](no1901877P.html) for details

### InterpreterRuntime::ldc()
See: [here](no1901803z.html) for details

### InterpreterRuntime::resolve_ldc()
See: [here](no190183Ix.html) for details
### Bytecode_loadconstant::resolve_constant()
(#Under Construction)







