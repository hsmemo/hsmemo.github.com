---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/objectMonitor.cpp
### 説明(description)

```
// complete_exit exits a lock returning recursion count
// complete_exit/reenter operate as a wait without waiting
// complete_exit requires an inflated monitor
// The _owner field is not always the Thread addr even with an
// inflated monitor, e.g. the monitor can be inflated by a non-owning
// thread due to contention.
```

### 名前(function name)
```
intptr_t ObjectMonitor::complete_exit(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	   Thread * const Self = THREAD;
	   assert(Self->is_Java_thread(), "Must be Java thread!");
	   JavaThread *jt = (JavaThread *)THREAD;
	
  {- -------------------------------------------
  (1) ObjectMonitor::DeferredInitialize() を呼んで, 
      (まだ初期化が終わっていなければ) Knob_* 変数の値の初期化を行う.
      ---------------------------------------- -}

	   DeferredInitialize();
	
  {- -------------------------------------------
  (1) もし ObjectMonitor 内の _owner フィールドがカレントスレッドを指していない場合, 
      _owner をカレントスレッドに変更しておく.
  
      (stack-locked 状態のロックが inflate された直後は, _owner に正しい値が入っていない.
       この状態になるのは, ロックを取った時には stack-locked だったものが, 
       開放までの間に何らかの理由で inflate された場合.
       つまりこの場合は, ロックは持っているが単に _owner に書かれていないというだけの状態.
       See: ObjectSynchronizer::inflate())
  
      (一応, ちゃんとロックを持っていること(= is_lock_owned() が true になること)を確認してはいる.
       が, ロックを持っていないケースはあり得ないはず.
       というか, その場合は下の guarantee で fail する.)
      ---------------------------------------- -}

	   if (THREAD != _owner) {
	    if (THREAD->is_lock_owned ((address)_owner)) {
	       assert(_recursions == 0, "internal state error");
	       _owner = THREAD ;   /* Convert from basiclock addr to Thread addr */
	       _recursions = 0 ;
	       OwnerIsThread = 1 ;
	    }
	   }
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	   guarantee(Self == _owner, "complete_exit not owner");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	   intptr_t save = _recursions; // record the old recursion count

  {- -------------------------------------------
  (1) (ロックを解放するので) _recursions は 0 に戻しておく.
      ---------------------------------------- -}

	   _recursions = 0;        // set the recursion level to be 0

  {- -------------------------------------------
  (1) ObjectMonitor::exit() を呼んで, ロックを解放する.
      ---------------------------------------- -}

	   exit (Self) ;           // exit the monitor

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	   guarantee (_owner != Self, "invariant");

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	   return save;
	}
	
```


