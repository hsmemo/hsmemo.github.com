---
layout: default
title: Thread の待機処理の枠組み ： 概要
---
[Up](noIpUCxk3g.html) [Top](../index.html)

#### Thread の待機処理の枠組み ： 概要

--- 
HotSpot 内には, スレッドの待機処理(スレッドをブロックさせる処理)の枠組みとして, 以下の 4種類が用意されている.

  * ParkEvent, os::PlatformEvent
  * SpinLock, Mux
  * Mutex, Monitor, MutexLocker, ...
  * ThreadCritical

### ParkEvent, os::PlatformEvent
最もプリミティブな要素.

OS レベルでのスケジューリング制御(pthread_cond_wait(), WaitForSingleObject(), 等)を隠蔽するレイヤー.

### SpinLock, Mux
ParkEvent を用いて構築されたシンプルなスピンロック, 及び Mutex 実装.

これらは, ごく短い critical section に対してのみ使われることになっている.

### Mutex, Monitor
ParkEvent 及び SpinLock, Mux を用いて構築された Mutex 及びモニタ実装.

長い critical section にも使用可能.

### ThreadCritical
非常に簡単な critical section に対してのみ使用される Mutex 実装.

HotSpot の初期化作業のかなり早い段階でも使われるため, 
ParkEvent (os::PlatformEvent) は使用せず独自の排他処理を行っている.

なお, 他の待機処理用のクラスに比べて使用箇所は少ない.







