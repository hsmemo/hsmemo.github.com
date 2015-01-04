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

```
TemplateTable::_new() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::_new() で処理を行う
       -> InterpreterRuntime::_new()
          -> ランタイムで処理を行う(下記参照)
```

* anewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

```
TemplateTable::anewarray() が生成するコード
-> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得.
       また現在実行中のメソッドに対応した constant pool を取得.
       -> InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
       -> InterpreterMacroAssembler::get_constant_pool() が生成するコード
   (1) InterpreterRuntime::anewarray() で処理を行う
       -> InterpreterRuntime::anewarray()
          -> ランタイムで処理を行う(下記参照)
```

* multianewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

```
TemplateTable::multianewarray() が生成するコード
-> InterpreterRuntime::multianewarray()
   -> (下記参照)
```

* getfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
* 1 回目
  TemplateTable::getfield() が生成するコード
  -> TemplateTable::getfield_or_static() が生成するコード
     -> (1) constant pool cache entry を取得
            -> TemplateTable::resolve_cache_and_index() が生成するコード
               -> (1) 既に作成済みであれば, それを取得
                      -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う (この場合はフィールドアクセスなので, InterpreterRuntime::resolve_get_put() を呼び出す)
                      -> InterpreterRuntime::resolve_get_put()
                         -> ランタイムで処理を行う(下記参照)
                  (3) まだ作成されて無かった場合は, ここで取得
                      -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) constant pool cache entry の中身を取り出す
            -> TemplateTable::load_field_cp_cache_entry() が生成するコード
   
* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -> (1) constant pool cache entry を取得
         -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
```

* getstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
TemplateTable::getstatic() が生成するコード
-> TemplateTable::getfield_or_static() が生成するコード
   -> (同上)
```

* putfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
* 1 回目
  TemplateTable::putfield() が生成するコード
  -> TemplateTable::putfield_or_static() が生成するコード
     -> (1) constant pool cache entry を取得
            -> TemplateTable::resolve_cache_and_index() が生成するコード
               -> (同上)
   
        (1) constant pool cache entry の中身を取り出す
            -> TemplateTable::load_field_cp_cache_entry() が生成するコード
   
* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -> (1) constant pool cache entry を取得
         -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
```

* putstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
TemplateTable::putstatic() が生成するコード
-> TemplateTable::putfield_or_static() が生成するコード
   -> (同上)
```

* invokevirtual 命令の処理  (See: [here](no3059-tw.html) for details)

```
TemplateTable::invokevirtual() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -> TemplateTable::resolve_cache_and_index() が生成するコード
             -> (1) 既に作成済みであれば, それを取得
                    -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic 以外の invoke* なので, InterpreterRuntime::resolve_invoke() を呼び出す)
                    -> InterpreterRuntime::resolve_invoke()
                       -> ランタイムで処理を行う(下記参照)
                (3) まだ作成されて無かった場合は, ここで取得
                    -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```

* invokeinterface 命令の処理  (See: [here](no3059L42.html) for details)

```
TemplateTable::invokeinterface() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -> (同上)
```

* invokestatic 命令の処理  (See: [here](nozMknLl1S.html) for details)

```
TemplateTable::invokestatic() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -> (同上)
```

* invokespecial 命令の処理  (See: [here](noZ4XAqR7E.html) for details)

```
TemplateTable::invokespecial() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -> (同上)
```

* invokedynamic 命令の処理 (See: [here](noikurSEOv.html) for details)

```
TemplateTable::invokedynamic() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
          -> TemplateTable::resolve_cache_and_index() が生成するコード
             -> (1) 既に作成済みであれば, それを取得
                    -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic なので, InterpreterRuntime::resolve_invokedynamic() を呼び出す)
                    -> InterpreterRuntime::resolve_invokedynamic()
                       -> ランタイムで処理を行う(下記参照)
                (3) まだ作成されて無かった場合は, ここで取得
                    -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```

* checkcast 命令の処理 (See: [here](no5F7e42op.html) for details)

```
TemplateTable::checkcast() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -> InterpreterRuntime::quicken_io_cc()
          -> ランタイムで処理を行う(下記参照)
```

* instanceof 命令の処理 (See: [here](no5F7e42op.html) for details)

```
TemplateTable::instanceof() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -> InterpreterRuntime::quicken_io_cc()
          -> ランタイムで処理を行う(下記参照)
```

* ldc, ldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

```
TemplateTable::ldc() が生成するコード
-> (1) constant pool をチェックして, ロード対象が未解決の文字列もしくは未解決のクラスかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 1byte/2byte を取得する
              -> ldub() が生成するコード (ldc の場合)
              -> InterpreterMacroAssembler::get_2_byte_integer_at_bcp() が生成するコード (ldc_w の場合)
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 対象が解決済みかどうかを調べる

   (1) 対象が未解決の文字列／クラスであれば InterpreterRuntime::ldc() で処理を行う
       -> InterpreterRuntime::ldc()
          -> ランタイムで処理を行う(下記参照)
```

* fast_aldc, fast_aldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

```
TemplateTable::fast_aldc() が生成するコード
-> TemplateTable::resolve_cache_and_index() が生成するコード
   -> (1) 既に作成済みであれば, それを取得
          -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
      (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は fast_aldc なので, InterpreterRuntime::resolve_ldc() を呼び出す)
          -> InterpreterRuntime::resolve_ldc()
             -> ランタイムで処理を行う(下記参照)
      (3) まだ作成されて無かった場合は, ここで取得
          -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```

#### x86-64 の場合
* new 命令の処理 (See: [here](no28916Rgx.html) for details)

```
TemplateTable::_new() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::_new() で処理を行う
       -> InterpreterRuntime::_new()
          -> ランタイムで処理を行う(下記参照)
```

* anewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

```
TemplateTable::anewarray() が生成するコード
-> (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得.
       また現在実行中のメソッドに対応した constant pool を取得.
       -> InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
       -> InterpreterMacroAssembler::get_constant_pool() が生成するコード
   (1) InterpreterRuntime::anewarray() で処理を行う
       -> InterpreterRuntime::anewarray()
          -> ランタイムで処理を行う(下記参照)
```

* multianewarray 命令の処理 (See: [here](no28916Rgx.html) for details)

```
TemplateTable::multianewarray() が生成するコード
-> InterpreterRuntime::multianewarray()
   -> ランタイムで処理を行う(下記参照)
```

* getfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
* 1 回目
  TemplateTable::getfield() が生成するコード
  -> TemplateTable::getfield_or_static() が生成するコード
     -> (1) constant pool cache entry を取得
            -> TemplateTable::resolve_cache_and_index() が生成するコード
               -> (1) 既に作成済みであれば, それを取得
                      -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                  (2) まだ作成されて無かった場合は, 作成処理を行う (この場合はフィールドアクセスなので, InterpreterRuntime::resolve_get_put() を呼び出す)
                      -> InterpreterRuntime::resolve_get_put()
                         -> ランタイムで処理を行う(下記参照)
                  (3) まだ作成されて無かった場合は, ここで取得
                      -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
   
        (1) constant pool cache entry の中身を取り出す
            -> TemplateTable::load_field_cp_cache_entry() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_accessfield() が生成するコード
  -> (1) constant pool cache entry を取得
         -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
```

* getstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
TemplateTable::getstatic() が生成するコード
-> TemplateTable::getfield_or_static() が生成するコード
   -> (同上)
```

* putfield 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
* 1 回目
  TemplateTable::putfield() が生成するコード
  -> TemplateTable::putfield_or_static() が生成するコード
     -> (1) constant pool cache entry を取得
            -> TemplateTable::resolve_cache_and_index() が生成するコード
               -> (同上)
   
        (1) constant pool cache entry の中身を取り出す
            -> TemplateTable::load_field_cp_cache_entry() が生成するコード

* 2 回目 (書き換え後)
  TemplateTable::fast_storefield() が生成するコード
  -> (1) constant pool cache entry を取得
         -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード

     (1) constant pool cache entry の中身を取り出す
```

* putstatic 命令の処理 (See: [here](nomIIS45Yh.html) for details)

```
TemplateTable::putstatic() が生成するコード
-> TemplateTable::putfield_or_static() が生成するコード
   -> (同上)
```

* invokevirtual 命令の処理  (See: [here](no3059-tw.html) for details)

```
TemplateTable::invokevirtual() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::prepare_invoke() が生成するコード
          -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
             -> TemplateTable::resolve_cache_and_index() が生成するコード
                -> (1) 既に作成済みであれば, それを取得
                       -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                   (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic 以外の invoke* なので, InterpreterRuntime::resolve_invoke() を呼び出す)
                       -> InterpreterRuntime::resolve_invoke()
                          -> ランタイムで処理を行う(下記参照)
                   (3) まだ作成されて無かった場合は, ここで取得
                       -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```

* invokeinterface 命令の処理  (See: [here](no3059L42.html) for details)

```
TemplateTable::invokeinterface() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::prepare_invoke() が生成するコード
          -> (同上)
```

* invokestatic 命令の処理  (See: [here](nozMknLl1S.html) for details)

```
TemplateTable::invokestatic() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::prepare_invoke() が生成するコード
          -> (同上)
```

* invokespecial 命令の処理  (See: [here](noZ4XAqR7E.html) for details)

```
TemplateTable::invokespecial() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::prepare_invoke() が生成するコード
          -> (同上)
```

* invokedynamic 命令の処理 (See: [here](noikurSEOv.html) for details)

```
TemplateTable::invokedynamic() が生成するコード
-> (1) constant pool cache entry を取得し, 中身を取り出す
       -> TemplateTable::prepare_invoke() が生成するコード
          -> TemplateTable::load_invoke_cp_cache_entry() が生成するコード
             -> TemplateTable::resolve_cache_and_index() が生成するコード
                -> (1) 既に作成済みであれば, それを取得
                       -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
                   (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は invokedynamic なので, InterpreterRuntime::resolve_invokedynamic() を呼び出す)
                       -> InterpreterRuntime::resolve_invokedynamic()
                          -> ランタイムで処理を行う(下記参照)
                   (3) まだ作成されて無かった場合は, ここで取得
                       -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```

* checkcast 命令の処理 (See: [here](no5F7e42op.html) for details)

```
TemplateTable::checkcast() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -> InterpreterRuntime::quicken_io_cc()
          -> ランタイムで処理を行う(下記参照)
```

* instanceof 命令の処理 (See: [here](no5F7e42op.html) for details)

```
TemplateTable::instanceof() が生成するコード
-> (1) constant pool をチェックして, 既に解決結果で書き換え済みかどうかを確認する.
       -> (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) 現在のバイトコード内から constant pool の index を示す 2byte を取得する
              -> InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード
          (1) index で tag を引いてみて, 解決結果が constant pool 中に書き込まれているかどうかを調べる
              (既に書き込まれていればそれを使うだけでいい)

   (1) まだ書き換えられていなければ InterpreterRuntime::quicken_io_cc() で処理を行う
       -> InterpreterRuntime::quicken_io_cc()
          -> ランタイムで処理を行う(下記参照)
```

* ldc, ldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

```
TemplateTable::ldc() が生成するコード
-> (1) constant pool をチェックして, ロード対象が未解決の文字列もしくは未解決のクラスかどうかを確認する.
       -> (1) 現在のバイトコード内から constant pool の index を示す 1byte/2byte を取得する
              -> ldub() が生成するコード (ldc の場合)
              -> InterpreterMacroAssembler::get_unsigned_2_byte_index_at_bcp() が生成するコード (ldc_w の場合)
          (1) 現在実行中のメソッドに対応した constant pool とその tag 配列を取得する
              -> InterpreterMacroAssembler::get_cpool_and_tags() が生成するコード
          (1) index で tag を引いてみて, 対象が解決済みかどうかを調べる

   (1) 対象が未解決の文字列／クラスであれば InterpreterRuntime::ldc() で処理を行う
       -> InterpreterRuntime::ldc()
          -> ランタイムで処理を行う(下記参照)
```

* fast_aldc, fast_aldc_w 命令の処理 (See: [here](nol1OD2rml.html) for details)

```
TemplateTable::fast_aldc() が生成するコード
-> TemplateTable::resolve_cache_and_index() が生成するコード
   -> (1) 既に作成済みであれば, それを取得
          -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
      (2) まだ作成されて無かった場合は, 作成処理を行う (この場合は fast_aldc なので, InterpreterRuntime::resolve_ldc() を呼び出す)
          -> InterpreterRuntime::resolve_ldc()
             -> ランタイムで処理を行う(下記参照)
      (3) まだ作成されて無かった場合は, ここで取得
          -> InterpreterMacroAssembler::get_cache_and_index_at_bcp() が生成するコード
```


### C++ Interpreter の場合
```
BytecodeInterpreter::run() もしくは BytecodeInterpreter::runWithChecks()
-> * new の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::_new()
              -> ランタイムで処理を行う(下記参照)

   * anewarray の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::anewarray()
              -> ランタイムで処理を行う(下記参照)

   * multianewarray の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::multianewarray()
              -> ランタイムで処理を行う(下記参照)

   * getfield, putfield, getstatic, putstatic の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::resolve_get_put()
              -> ランタイムで処理を行う(下記参照)

   * invokeinterface, invokevirtual, invokespecial, invokestatic の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::resolve_invoke()
              -> ランタイムで処理を行う(下記参照)

   * invokedynamic の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::resolve_invokedynamic()
              -> ランタイムで処理を行う(下記参照)

   * instanceof, checkcast の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::quicken_io_cc()
              -> ランタイムで処理を行う(下記参照)

   * ldc, ldc_w の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::ldc()
              -> ランタイムで処理を行う(下記参照)

   * fast_aldc, fast_aldc_w の場合
     -> CALL_VM() マクロ
        -> CALL_VM_NOCHECK() マクロ
           -> InterpreterRuntime::resolve_ldc()
              -> ランタイムで処理を行う(下記参照)
```

### 両 Interpreter 共通の処理 (InterpreterRuntime の処理)
* new 命令の処理 (See: [here](no28916DqA.html) for details)

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::_new()
   -> (1) 対象のクラスの解決処理を行う
          -> constantPoolOopDesc::klass_at()
             -> (See: [here](no-VW6a46T.html) for details)
```

* anewarray 命令の処理 (See: [here](no28916DqA.html) for details)

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::anewarray()
   -> (1) 対象のクラスの解決処理を行う
          -> constantPoolOopDesc::klass_at()
             -> (See: [here](no-VW6a46T.html) for details)
```

* multianewarray 命令の処理 (See: [here](no28916DqA.html) for details)

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::multianewarray()
   -> (1) 対象のクラスの解決処理を行う
          -> constantPoolOopDesc::klass_at()
             -> (See: [here](no-VW6a46T.html) for details)
```

* getfield, getstatic, putfield, putstatic 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::resolve_get_put()
   -> (1) 対象のフィールド名の解決を行う
          -> LinkResolver::resolve_field()
             -> (See: [here](no-VW6a46T.html) for details)

      (1) 解決結果を constant pool cache entry にキャッシュしておく
          -> ConstantPoolCacheEntry::set_field()
```

* invokevirtual, invokeinterface, invokespecial, invokestatic 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::resolve_invoke()
   -> (1) 対象のメソッド名の解決を行う
          -> LinkResolver::resolve_invoke()
             -> (See: [here](no-VW6a46T.html) for details)

      (1) 解決結果を constant pool cache entry にキャッシュしておく
          -> ConstantPoolCacheEntry::set_method()         (invokeinterface 以外の場合(※))
          -> ConstantPoolCacheEntry::set_interface_call() (invokeinterface の場合(※))

(※) ただし, invokeinterface であっても 
     java.lang.Object のメソッドの場合には ConstantPoolCacheEntry::set_method() を使用する.
     (これは Java 仮想マシン仕様のコーナーケースとのこと #TODO)
```

* invokedynamic 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::resolve_invokedynamic()
   -> #TODO
```

* checkcast, instanceof 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::quicken_io_cc()
   -> (1) 対象のクラスの解決処理を行う
          -> constantPoolOopDesc::klass_at()
             -> (See: [here](no-VW6a46T.html) for details)
```

* ldc, ldc_w 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::ldc()
   -> * ロード対象がクラスオブジェクトの場合:
        -> constantPoolOopDesc::klass_at()
           -> (See: [here](no-VW6a46T.html) for details)
      * ロード対象が文字列オブジェクトの場合:
        -> constantPoolOopDesc::string_at()
           -> (See: [here](no-VW6a46T.html) for details)
```

* fast_aldc, fast_aldc_w 命令の処理

```
各インタープリタの処理から呼び出される(上記参照)
-> InterpreterRuntime::resolve_ldc()
   -> Bytecode_loadconstant::resolve_constant()
      -> 
```


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







