---
layout: default
title: Runtime による Thread の一時停止処理 (VM Operation) 
---
[Up](noJb9iXZL-.html) [Top](../index.html)

#### Runtime による Thread の一時停止処理 (VM Operation) 

--- 
## 概要(Summary)
VM Operation は, 専用のスレッド ("VMThread") によって実行される.

VM Operation処理は, VMThread::execute() メソッドによって VM Operation 要求が VMThread に通知されることで開始される.
処理の流れは以下のようになる.

1. 要求を出される VMThread 自身は, HotSpot の起動時に作成された後, 
   要求が来るまで無限ループで待機している.

2. VMThread::execute() が呼び出されると,
   VMThread::_vm_queue という VMOperationQueue に VM Operation 要求が追加される.
   さらに VMOperationQueue_lock に notify() が通知される.
   
   この notify() によって待機していた VMThread が起床し,
   VMThread::_vm_queue から VM Operation オブジェクトを取得し, 
   VMThread::evaluate_operation() で VM Operation を実行する.




## Subcategories
* [Runtime による Thread の一時停止処理 ： VMThread の生成処理](no-la6kE9R.html)
* [Runtime による Thread の一時停止処理 ： VM Operation の起動処理  ](no2935qaz.html)



