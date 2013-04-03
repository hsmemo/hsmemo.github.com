---
layout: default
title: Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： ランタイムフレーム → 非ランタイム(かつ非ネイティブ)フレームの境界での処理  
---
[Up](noAJsAY6Zl.html) [Top](../index.html)

#### Exception の処理 ： 処理の詳細 (4) ： 例外ハンドラの探索処理 ： 異種フレーム境界での処理 ： ランタイムフレーム → 非ランタイム(かつ非ネイティブ)フレームの境界での処理  

--- 
## 概要(Summary)
例外が非ランタイム(かつ非ネイティブ)フレーム内で伝搬され続け, ランタイムフレームまで戻ってきた場合, 
その境界で (非ランタイムフレーム用のものからランタイムフレーム用のものへと) 例外ハンドラ探索処理が切り替わる.

切り替えは, 呼び出したランタイム側がリターン時に CHECK マクロ等でチェックすることで行っている
(ランタイムからの呼び出しに使われる JavaCalls::call_*() 等の中では CHECK マクロが使われているため,
 普通に pending_exception をセットした状態でリターンすれば, 
 後はランタイム内で勝手に例外が持ち上げられていく. (See: [here](no3059iJu.html) for details)).

以上のことから, この境界では普通にリターンするだけで例外が伝搬される.
ただし, 呼び出し元に例外情報を伝えるためには 
JavaThread 内の例外関係のフィールドを設定しておく必要がある.
このため, その処理を StubRoutines::catch_exception_entry() が指しているコードで行ってからリターンしている.

## 備考(Notes)
なお, どこまで遡ってもランタイムフレームがないということはあり得ない.
どんなスレッドも最初にランタイムが JavaCalls で Java のメソッドを呼び出す所から始まるので.

## 処理の流れ (概要)(Execution Flows : Summary)
```
各種のスタックの unwind 処理 (See: [here](noQ5C359iU.html) for details)
-> StubRoutines::catch_exception_entry() が指しているコード (= StubGenerator::generate_catch_exception() が生成したコード)
   -> JavaThread 内の例外関係のフィールドを設定
   -> StubRoutines::_call_stub_return_address が指しているアドレスにジャンプ 
      (リターンとほぼ同義. スタックフレームが既に無いので単にジャンプする)
```


## 処理の流れ (詳細)(Execution Flows : Details)
### StubGenerator::generate_catch_exception() (sparc の場合)
See: [here](no3059CTn.html) for details
### StubGenerator::generate_catch_exception() (x86_64 の場合)
See: [here](no3059Pdt.html) for details






