---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： HotSpot 内へのリターン時
---
[Up](nouSkNo9hy.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： ネイティブコードの処理中 ： HotSpot 内へのリターン時

--- 
## 概要(Summary)
ネイティブコードから HotSpot 内に戻ってきた際には, 明示的に SafepointSynchronize::_state のチェックが行われる. 

この時点で Safepoint が開始されていた場合, そのスレッドはその場で停止する.

## 処理の流れ (概要)(Execution Flows : Summary)
### ネイティブメソッド呼び出し処理中での確認処理
```
InterpreterGenerator::generate_native_entry()  or  SharedRuntime::generate_native_wrapper()  が生成したコード
-> (1) JavaThreadState を _thread_in_native_trans に変更
   (1) メモリアクセスの順序づけを行う
       -> MacroAssembler::membar() or MacroAssembler::serialize_memory() が生成したコード
   (1) SafepointSynchronize::_state の値を確認
```

### 上記確認処理でメモリアクセス違反が生じた場合の処理
```
シグナルハンドラ (JVM_handle_linux_signal() or JVM_handle_solaris_signal() or topLevelExceptionFilter()) (See: [here](noNmlmYDJk.html) for details)
-> (1) os::serialize_thread_states() の処理との同期を取る
       -> os::block_on_serialize_page_trap()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### InterpreterGenerator::generate_native_entry() (Sparc の場合)
See: [here](no3718Gvm.html) for details
(#Under Construction)

### InterpreterGenerator::generate_native_entry() (x86_64 の場合)
(#Under Construction)

### SharedRuntime::generate_native_wrapper() (Sparc の場合)
(#Under Construction)

### SharedRuntime::generate_native_wrapper() (x86_64 の場合)
(#Under Construction)

### os::block_on_serialize_page_trap()
See: [here](no7882G-o.html) for details






