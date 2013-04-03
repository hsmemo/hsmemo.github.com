---
layout: default
title: Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： 共通処理
---
[Up](noadKcOM5n.html) [Top](../index.html)

#### Thread の一時停止処理の枠組み (Safepoint 処理) ： 停止される側の処理 ： 共通処理

--- 
## 概要(Summary)
停止処理が行われる処理パスは沢山存在する.

ただし停止処理自体は SafepointSynchronize::block() で実装されており, どの場合もこれを呼び出すことで停止する.

## 備考(Notes)
SafepointSynchronize::do_call_back() で,
明示的に SafepointSynchronize::_state をチェックできる.

(SafepointSynchronize::_state は, 非 Safepoint 時には _not_synchronized という値をしている.
 safepoint が開始されると, _synchronizing という値になる.
 全スレッドが停止し終わると _synchronized になる)

ついでに, SafepointSynchronize::is_at_safepoint() という類似品もあるが, こっちは assert 内でしか使われていない模様(?)

## 処理の流れ (概要)(Execution Flows : Summary)
```
(種々の状態遷移処理)
-> SafepointSynchronize::block()
   -> (1) 処理対象のスレッドが既に終了していれば, ここでリターンする or 永久にブロックする
          (HotSpot の終了処理が始まったことでスレッドが終了した場合は永久にブロック)
          -> JavaThread::block_if_vm_exited()

      (1) スタックフレームを辿れるようにしておく
          -> JavaFrameAnchor::make_walkable()

      (1) JavaThreadState を _thread_blocked に変更する.
          -> JavaThread::set_thread_state()

      (1) 対象のスレッドの JavaThreadState が _thread_in_vm_trans または _thread_in_Java であり,
          かつ既に VMThread による Safepoint 処理が開始されており, さらに自分が最後のスレッドの場合は, VM Thread を起こしておく.
          -> Monitor::notify_all()

      (1) ブロックする
          -> Monitor::lock_without_safepoint_check()

      (1) JavaThreadState を元に戻す.
          -> JavaThread::set_thread_state()

(明示的な safepoint チェック処理)
-> SafepointSynchronize::do_call_back()
```

## 処理の流れ (詳細)(Execution Flows : Details)
### SafepointSynchronize::block()
See: [here](no7882E8a.html) for details
### JavaThread::block_if_vm_exited()
See: [here](no78825zi.html) for details
### JavaFrameAnchor::make_walkable()
(#Under Construction)

### JavaThread::set_thread_state()
See: [here](no31977pVc.html) for details
### SafepointSynchronize::do_call_back()
See: [here](no7882fRu.html) for details






