---
layout: default
title: Template Interpreter によるバイトコードの実行処理 ： テンプレートの生成処理
---
[Up](noaqS079AL.html) [Top](../index.html)

#### Template Interpreter によるバイトコードの実行処理 ： テンプレートの生成処理

--- 
## 概要(Summary)
各テンプレートは TemplateInterpreterGenerator::generate_and_dispatch() で生成される.
TemplateInterpreterGenerator::generate_and_dispatch() の処理は大別すると 3つのフェーズに分けられる.
このため, 生成されるコード(テンプレート)も大きく分けると 3つの部位からなる.

  1. まず, テンプレートの入り口部分用のコードを生成 (全テンプレートで共通の部分)    (InterpreterMacroAssembler::dispatch_prolog()),
  2. 次に, バイトコード本体の処理を行うコードを生成 (テンプレート毎に異なる部分)     (Template::generate()),
  3. 最後に, 次のバイトコードにジャンプするコードを生成 (全テンプレートで共通の部分) (InterpreterMacroAssembler::dispatch_epilog())

TemplateInterpreterGenerator::generate_and_dispatch() によるテンプレート生成処理は, HotSpot 起動中に TemplateInterpreterGenerator::set_entry_points_for_all_bytes() 内で呼び出される.
この際には, 以下の 3つの生成関数のどれかを経由する
(どの関数も最終的には TemplateInterpreterGenerator::generate_and_dispatch() を呼び出す).
どれを経由するかは2つの条件に応じて決められる (対象のバイトコードが wide prefix 付きか否か, TOS の想定が vtos のみか否か).

  * TemplateInterpreterGenerator::set_short_entry_points()

    非 wide 命令用 (short 命令用).
    TOS 状態に応じて処理を分けるコードを生成する
    (vtos であれば pop を行う処理パスを生成し, 
    対応する TOS であればいきなりコードに入る処理パスを生成する).

  * TemplateInterpreterGenerator::set_vtos_entry_points()

    非 wide 命令用 (short 命令用).
    エントリ時の TOS 状態が vtos でしかあり得ない場合に使用される最適化版.

  * TemplateInterpreterGenerator::set_wide_entry_point()

    wide 命令用.
    現在では vtos しか許して無いため, 単に generate_and_dispatch() でコード生成するだけ.


## 備考(Notes)
* 生成されたテンプレートの各エントリアドレスは dispatch table 中の対応箇所に格納される.
  
  なお dispatch table は, 
  非 wide 命令については Interpreter::_normal_table, 
  wide 命令に付いては Interpreter::_wentry_point.

* dispatch table 中の不正なエントリポイントは, 使われるとエラーが起こるようになっている.
  
  初期状態では, dispatch table 中の各 TOS 用のエントリポイントアドレスは, 
  _illegal_bytecode_sequence や _unimplemented_bytecode になっている.
  
  そして, dispatch table にエントリポイントを設定する処理でも, 
  正常箇所しか設定は行われないため, 不正な状態の TOS 用のエントリポイントはそのまま残される.

## 処理の流れ (概要)(Execution Flows : Summary)
```
(See: [here](no3059SwU.html) for details)
-> TemplateInterpreterGenerator::set_entry_points_for_all_bytes()
   -> * bytecode として無効なエントリの場合:
        -> TemplateInterpreterGenerator::set_unimplemented()  (<= 異常終了するコードを生成)
      * bytecode として有効なエントリの場合:
        -> TemplateInterpreterGenerator::set_entry_points()
           -> * wide prefix 付きではないバイトコードの場合:
                -> TemplateInterpreterGenerator::set_short_entry_points()
                   -> * そのバイトコードが想定している TOS 状態が vtos の場合:
                        -> TemplateInterpreterGenerator::set_vtos_entry_points()
                           -> TemplateInterpreterGenerator::generate_and_dispatch()
                              -> (1) 必要に応じてデバッグ用のコードを生成
                                     -> TemplateInterpreterGenerator::histogram_bytecode()
                                     -> TemplateInterpreterGenerator::count_bytecode()
                                     -> TemplateInterpreterGenerator::histogram_bytecode_pair()
                                     -> TemplateInterpreterGenerator::trace_bytecode()
                                     -> TemplateInterpreterGenerator::stop_interpreter_at()
                            
                                 (1) テンプレート入り口用のコードを生成.
                                     -> InterpreterMacroAssembler::dispatch_prolog()
                                        -> (See: [here](noEgWr8prQ.html) for details)
                            
                                 (1) バイトコード本体の処理を行うコードを生成
                                     -> Template::generate()
                                        -> (テンプレート毎に異なる生成関数を呼び出す)
                                           (See: )
                            
                                 (1) テンプレート終了部用のコード(次のバイトコードに遷移する処理)を生成.
                                     -> InterpreterMacroAssembler::dispatch_epilog()
                                        -> (See: [here](noEgWr8prQ.html) for details)

                      * それ以外の場合:
                        -> TemplateInterpreterGenerator::generate_and_dispatch()
                           -> (同上)
              * wide prefix 付きのバイトコードの場合:
                -> TemplateInterpreterGenerator::set_wide_entry_point()
                   -> TemplateInterpreterGenerator::generate_and_dispatch()
                      -> (同上)
```

## 処理の流れ (詳細)(Execution Flows : Details)
### TemplateInterpreterGenerator::set_entry_points_for_all_bytes()
See: [here](no7882VYl.html) for details
### TemplateInterpreterGenerator::set_entry_points()
See: [here](no7882iir.html) for details
### TemplateInterpreterGenerator::set_short_entry_points()
See: [here](no7882vsx.html) for details
### TemplateInterpreterGenerator::set_wide_entry_point()
See: [here](no7882h2A.html) for details
### TemplateInterpreterGenerator::set_vtos_entry_points()  (sparc の場合)
See: [here](no7882uAH.html) for details
### TemplateInterpreterGenerator::set_vtos_entry_points()  (x86_64 の場合)
See: [here](no78827KN.html) for details
### TemplateInterpreterGenerator::generate_and_dispatch()
See: [here](no7882IVT.html) for details
### TemplateInterpreterGenerator::histogram_bytecode()  (sparc の場合)
(#Under Construction)
### TemplateInterpreterGenerator::histogram_bytecode()  (x86_64 の場合)
(#Under Construction)
### TemplateInterpreterGenerator::count_bytecode()  (sparc の場合)
(#Under Construction)
### TemplateInterpreterGenerator::count_bytecode()  (x86_64 の場合)
(#Under Construction)
### TemplateInterpreterGenerator::histogram_bytecode_pair()  (sparc の場合)
(#Under Construction)
### TemplateInterpreterGenerator::histogram_bytecode_pair()  (x86_64 の場合)
(#Under Construction)
### TemplateInterpreterGenerator::trace_bytecode()  (sparc の場合)
(#Under Construction)
### TemplateInterpreterGenerator::trace_bytecode()  (x86_64 の場合)
(#Under Construction)
See: [here](no2747hZB.html) for details
### TemplateInterpreterGenerator::stop_interpreter_at()  (sparc の場合)
(#Under Construction)
### TemplateInterpreterGenerator::stop_interpreter_at()  (x86_64 の場合)
(#Under Construction)

### InterpreterMacroAssembler::verify_method_data_pointer()  (sparc の場合)
See: [here](no7882VfZ.html) for details
### InterpreterMacroAssembler::verify_method_data_pointer()  (x86_64 の場合)
See: [here](no7882ipf.html) for details
### Template::generate()
See: [here](no7882IcH.html) for details
### TemplateInterpreterGenerator::set_unimplemented()
See: [here](no7882JPm.html) for details






