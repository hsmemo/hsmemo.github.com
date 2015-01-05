---
layout: default
title: Class のロード/リンク/初期化 ： リンク処理 (4) ： rewrite 処理 (& constant pool cache の作成) 
---
[Up](noX5hsnWQw.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： リンク処理 (4) ： rewrite 処理 (& constant pool cache の作成) 

--- 
## 概要(Summary)
バイトコード検証の完了後, 高速化のために一部のバイトコードを書き換える処理 (rewrite 処理) が行われる.
この処理は Rewriter::rewrite() で行われる.

また以上の rewrite 処理後には, jsr バイトコードに関する再配置処理と, 
各メソッドにエントリポイントを設定する処理が行われる.
この処理は Rewriter::relocate_and_link() で行われる.

なお, rewrite 処理で行われる処理は以下の通り.

  * java.lang.Object のコンストラクタ (= `<init>()` メソッド) 中の return 命令を, 
    HotSpot独自のバイトコード命令に変更する (return_register_finalizer 命令)
    
    これは Finalizer の処理を補佐するバイトコード命令 (See: [here](nocCkfpcQQ.html) for details).
    実行したオブジェクトを Finalizer に登録してからリターンする (See: [here](no30590Am.html) for details).

  * lookupswitch 命令を, HotSpot独自のバイトコード命令に変更する
    (fast_linearswitch 命令または fast_binaryswitch 命令).
    
    飛び先の候補数に応じた2種類が用意されており, 
    候補数が閾値(※)未満であれば fast_linearswitch, それ以上であれば fast_binaryswith になる.
    fast_linearswitch は線形にサーチし, fast_binaryswith はバイナリサーチを行う (See: [here](noS59wryRf.html) for details).
    
    (※) 閾値は develop オプションである BinarySwitchThreshold オプションの値. デフォルトでは 5.
    
  * ロード対象が method handle もしくは method type である
    ldc 命令及び ldc_w 命令を, HotSpot独自のバイトコード命令に変更する
    (fast_aldc 命令, fast_aldc_w 命令). (See: [here](nol1OD2rml.html) for details)

  * constant pool の index をオペランドとするバイトコード命令に対し, 
    index 部分を constant pool cache の index に置き換える
    
    (対象のバイトコード命令は,
    getstatic, getfield, putstatic, putfield,
    invokestatic, invokespecial, invokevirtual, invokeinterface, invokedynamic)
    
    なお, この時点では constantPoolCacheOopDesc の中は空であり,
    シンボルの解決時にそれぞれの中身が作成される (See: [here](no7882NqI.html) for details).

  * constant pool cache (constantPoolCacheOop オブジェクト) を作成する


## 備考(Notes)
以下のコマンドラインオプションが rewrite 処理に影響する.

* RegisterFinalizersAtInit
  
  `java.lang.Object.<init>()` 中の return 命令を書き換えるかどうかを制御する.
  オプションをオフにした場合は return 命令の書き換えは行わない.

  なお, オプションをオフにした場合は,
  オブジェクトのメモリを確保する際に (= instanceKlass::allocate_instance() 内で)
  Finalizer に登録している模様.
  (See: [here](no28916Q0G.html) for details)

* ...

## 処理の流れ (概要)(Execution Flows : Summary)
* Rewriter::rewrite() の処理

```
Rewriter::rewrite(instanceKlassHandle klass, TRAPS)
-> Rewriter::Rewriter()
   -> (1) 対象が java.lang.Object の場合は, コンストラクタ(<init>()) 中の return 命令を return_register_finalizer 命令に書き換える.
          (なお, RegisterFinalizersAtInit オプションがセットされていない場合は, この処理は行わない)
          -> Rewriter::rewrite_Object_init()

      (1) 処理対象クラス内の各メソッドに対して, 上記以外のバイトコード書き換え処理を行う.
          -> Rewriter::scan_method()
             -> 全てのメソッド内の各バイトコードに対して書き換えを行う.
                * lookupswitch 命令の場合:
                  -> fast_linearswitch 命令または fast_binaryswitch 命令に書き換える

                * fast_linearswitch 命令または fast_binaryswitch 命令の場合:
                  -> lookupswitch 命令に書き換える (元に戻す)

                * {get|put}{static|field} 命令や invoke* 命令(invokedynamic除く) の場合:
                  -> Rewriter::rewrite_member_reference()
                     (バイトコード命令中の Constant Pool index が埋まっている 2byte を 
                     Constant Pool Cache の index に書き換える)

                * invokedynamic 命令の場合:
                  -> Rewriter::rewrite_invokedynamic()
                     (バイトコード命令中の Constant Pool index が埋まっている 4byte を 
                     Constant Pool Cache の index に書き換える)

                * ldc 命令または ldc_w 命令の場合:
                  -> Rewriter::maybe_rewrite_ldc()
                     (ロード対象が method handle もしくは method type の場合, 
                     fast_aldc 命令または fast_aldc_w 命令に置き換える)

      (1) 収集した constant pool cache index の情報を元に constant pool cache (constantPoolCacheOop オブジェクト) を生成する.
          -> Rewriter::make_constant_pool_cache()
             -> oopFactory::new_constantPoolCache()
                 -> constantPoolCacheKlass::allocate()
```

* Rewriter::relocate_and_link() の処理

```
Rewriter::relocate_and_link(instanceKlassHandle this_oop, TRAPS)
-> Rewriter::relocate_and_link(instanceKlassHandle this_oop, objArrayHandle methods, TRAPS)
   -> (1) jsr バイトコードに関する再配置処理を行う
          -> Rewriter::rewrite_jsrs()
             -> ResolveOopMapConflicts::do_potential_rewrite()
                -> GenerateOopMap::compute_map()
                   -> 

      (1) 対象のメソッドにエントリポイントを設定する
          (これにより, このメソッドはインタープリタ／JIT生成コードから呼び出せるようになる)
          -> methodOopDesc::link_method()
             -> (See: [here](no7882a7C.html) for details)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### Rewriter::rewrite(instanceKlassHandle klass, TRAPS)
See: [here](no18536H2Z.html) for details
### Rewriter::Rewriter()
See: [here](no18536H9N.html) for details
### Rewriter::compute_index_maps()
See: [here](no18536jM0.html) for details
### Rewriter::rewrite_Object_init()
See: [here](no18536MXN.html) for details
### Rewriter::scan_method()
See: [here](no18536q9d.html) for details
### Rewriter::rewrite_member_reference()
See: [here](no18536kWZ.html) for details
### Rewriter::rewrite_invokedynamic()
See: [here](no19018Lde.html) for details
### Rewriter::maybe_rewrite_ldc()
See: [here](no18536blL.html) for details
### Rewriter::make_constant_pool_cache()
See: [here](no19018xBe.html) for details
### Rewriter::relocate_and_link(instanceKlassHandle this_oop, TRAPS)
See: [here](no19018oXE.html) for details
### Rewriter::relocate_and_link(instanceKlassHandle this_oop, objArrayHandle methods, TRAPS)
See: [here](no19018cHR.html) for details
### Rewriter::rewrite_jsrs()
(#Under Construction)
See: [here](no19018ICN.html) for details
### ResolveOopMapConflicts::do_potential_rewrite()
(#Under Construction)
See: [here](no19018way.html) for details
### GenerateOopMap::compute_map()
See: [here](no2935NYj.html) for details

### methodOopDesc::link_method()
(#Under Construction)
See: [here](no19018rcD.html) for details






