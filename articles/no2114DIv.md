---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/mutex.cpp
### 説明(description)


```
// Polite TATAS spinlock with exponential backoff - bounded spin.
// Ideally we'd use processor cycles, time or vtime to control
// the loop, but we currently use iterations.
// All the constants within were derived empirically but work over
// over the spectrum of J2SE reference platforms.
// On Niagara-class systems the back-off is unnecessary but
// is relatively harmless.  (At worst it'll slightly retard
// acquisition times).  The back-off is critical for older SMP systems
// where constant fetching of the LockWord would otherwise impair
// scalability.
//
// Clamp spinning at approximately 1/2 of a context-switch round-trip.
// See synchronizer.cpp for details and rationale.
```

### 名前(function name)
```
int Monitor::TrySpin (Thread * const Self) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まずは Monitor::TryLock() でロック確保を試みる. 
      成功したらここでリターン (ロックが取れたので 1 をリターン).
      ---------------------------------------- -}

	  if (TryLock())    return 1 ;

  {- -------------------------------------------
  (1) マルチプロセッサ環境でなければ, スピンロックは無駄なだけなので, ここでリターン.
      (ロックは取れていないので 0 をリターン)
      ---------------------------------------- -}

	  if (!os::is_MP()) return 0 ;
	
  {- -------------------------------------------
  (1) (変数宣言など)
      (SpinMax は, スピンロックを試みる最大回数. ここまで試して駄目ならリターンする.
       
      ---------------------------------------- -}

	  int Probes  = 0 ;
	  int Delay   = 0 ;
	  int Steps   = 0 ;
	  int SpinMax = NativeMonitorSpinLimit ;
	  int flgs    = NativeMonitorFlags ;

  {- -------------------------------------------
  (1) 以下, CAS によるロックが成功するかスピンロックを諦めるまで, 次の for ループを繰り返す.
      スピンロックを諦めるのは以下の条件が成り立った場合.
      * スピンロックを試みた回数が SpinMax を超えた場合
      * Safepoint が開始されていた場合
      ---------------------------------------- -}

	  for (;;) {

    {- -------------------------------------------
  (1.1) cxq(_LockWord)がロックされていなければ (= ロックビット(_LBIT)が立っていなければ), 
        CASPTR でロックを試みる.
        成功したら, ここでリターン (ロックが取れたので 1 をリターン).
        失敗したら, ループの先頭に戻ってやり直し.
        ---------------------------------------- -}

	    intptr_t v = _LockWord.FullWord;
	    if ((v & _LBIT) == 0) {
	      if (CASPTR (&_LockWord, v, v|_LBIT) == v) {
	        return 1 ;
	      }
	      continue ;
	    }
	
    {- -------------------------------------------
  (1.1) SpinPause() を呼び出して, 少しの時間だけ待機してみる.
        (なお, この最適化(?)は NativeMonitorFlags の 4bit目(8)が立っていない場合にのみ行われる)
        ---------------------------------------- -}

	    if ((flgs & 8) == 0) {
	      SpinPause () ;
	    }
	
    {- -------------------------------------------
  (1.1) もしスピンロックを試みた回数が SpinMax を超えていれば, ここでリターン.
        (ロックは取れていないので 0 をリターン)
        ---------------------------------------- -}

	    // Periodically increase Delay -- variable Delay form
	    // conceptually: delay *= 1 + 1/Exponent
	    ++ Probes;
	    if (Probes > SpinMax) return 0 ;
	
    {- -------------------------------------------
  (1.1) 適当な回数(以下の Delay)分だけスピンループを行い, 少しの時間だけ待機してみる.
        (なお, この処理は NativeMonitorFlags の 2bit目(2)が立っていない場合にのみ行われる)
  
        スピンループ内では, MarsagliaXORV() (もしくは Stall()) を呼んで乱数生成を行っている.
        (この乱数生成はコンパイラによる最適化がされにくいので, それなりにきちんと時間がつぶせる模様)
  
        また, スピンループ回数(Delay)は, 初めは 1 から開始し, 7回ループする毎に以下のように増加させている.
          Delay = ((Delay << 1)|1) & 0x7FF ;
          // CONSIDER: Delay += 1 + (Delay/4); Delay &= 0x7FF ;
        
        (なお, スピンループ中では SafepointSynchronize::do_call_back() を呼んで 
         Safepoint が開始されているかどうかも確認しており, 
         もし Safepoint 中であれば即座にリターンするコードも入っている
         (この場合, ロックは取れていないので 0 をリターン).
         ただし, この確認処理＆リターン処理は NativeMonitorFlags の 3bit目(4)が立っていない場合にのみ行われる.)
  
        (ちなみにコメントでは, 
         ロックを握っているスレッドが稼働中かどうか(schedctl state)を確かめて, 
         稼働中でなければ(OFFPROCなら)スピンロックを止める, みたいな処理を入れてはどうか, 
         とも書かれている)
  
        (またコメントでは, スピンループに使用する命令の候補として以下のようなものが挙げられている.
           PAUSE, SLEEP, MEMBAR #sync, MEMBAR #halt,
           wr %g0,%asi, gethrtime, rdstick, rdtick, rdtsc, etc. ...
         極力, キャッシュのコンシステンシー制御のためのトラフィックが出ない方が嬉しいので, 
         store 命令とかは避けた方がいい.
         現状では, Marsaglia Shift-Xor RNG loop を使っている.)
        ---------------------------------------- -}

	    if ((Probes & 0x7) == 0) {
	      Delay = ((Delay << 1)|1) & 0x7FF ;
	      // CONSIDER: Delay += 1 + (Delay/4); Delay &= 0x7FF ;
	    }
	
	    if (flgs & 2) continue ;
	
	    // Consider checking _owner's schedctl state, if OFFPROC abort spin.
	    // If the owner is OFFPROC then it's unlike that the lock will be dropped
	    // in a timely fashion, which suggests that spinning would not be fruitful
	    // or profitable.
	
	    // Stall for "Delay" time units - iterations in the current implementation.
	    // Avoid generating coherency traffic while stalled.
	    // Possible ways to delay:
	    //   PAUSE, SLEEP, MEMBAR #sync, MEMBAR #halt,
	    //   wr %g0,%asi, gethrtime, rdstick, rdtick, rdtsc, etc. ...
	    // Note that on Niagara-class systems we want to minimize STs in the
	    // spin loop.  N1 and brethren write-around the L1$ over the xbar into the L2$.
	    // Furthermore, they don't have a W$ like traditional SPARC processors.
	    // We currently use a Marsaglia Shift-Xor RNG loop.
	    Steps += Delay ;
	    if (Self != NULL) {
	      jint rv = Self->rng[0] ;
	      for (int k = Delay ; --k >= 0; ) {
	        rv = MarsagliaXORV (rv) ;
	        if ((flgs & 4) == 0 && SafepointSynchronize::do_call_back()) return 0 ;
	      }
	      Self->rng[0] = rv ;
	    } else {
	      Stall (Delay) ;
	    }
	  }
	}
	
```


