---
layout: default
title: Thread の待機処理の枠組み ： ParkEvent (os：：PlatformEvent) による処理 
---
[Up](noIpUCxk3g.html) [Top](../index.html)

#### Thread の待機処理の枠組み ： ParkEvent (os：：PlatformEvent) による処理 

--- 
## 概要(Summary)
Thread の待機処理は 
ParkEvent クラス (およびそのスーパークラスの os::PlatformEvent クラス) が担当している.
実際の処理のほとんどは os::PlatformEvent クラスに実装されている. (See: ParkEvent, os::PlatformEvent).

ParkEvent クラスは以下のように使用する.

1. ParkEvent::Allocate() で, ParkEvent オブジェクトを生成する.

2. ParkEvent オブジェクトに対して ParkEvent::park() を呼ぶと, park() を呼んだスレッドがブロックされる.

   (なお正確に言うと ParkEvent クラスはこのメソッドをオーバーライドしていないので, 実際には os::PlatformEvent::park() のこと)

3. 別のスレッドがその ParkEvent オブジェクトに対して ParkEvent::unpark() を呼ぶと, ブロックしていたスレッドが起床される.

   (なお正確に言うと ParkEvent クラスはこのメソッドをオーバーライドしていないので, 実際には os::PlatformEvent::unpark() のこと)

4. なお, ブロックにタイムアウトを付けたい場合は, 
   ParkEvent::park() の代わりに ParkEvent::park(jlong millis) を使えばいい.

   (なお正確に言うと ParkEvent クラスはこのメソッドをオーバーライドしていないので, 実際には os::PlatformEvent::park(jlong millis) のこと)

5. 不要になった ParkEvent オブジェクトは, ParkEvent::Release() で解放する.

## 備考(Notes)
### Linux 上での実装について
コメントによると, NPTL で pthread_cond_timedwait() がバグっていた時期があったらしい
(永久にハングしてしまうんだとか).

以下の 4種類の解決策が考えられるが, 現在採用しているのは "4.". 
(WorkAroundNPTLTimedWaitHang オプションを参照).  
(<= なおコメント中の WorkAroundNTPLTimedWaitHang は typo(TとPが逆))

なお, 「使用しているカーネルや glibc のバージョンを確認して, 
バグっているバージョンの場合だけ対処する最適化も考えられる」とのこと.

1. ...
2. ...
3. ...
4. ...


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.cpp))
    // Refer to the comments in os_solaris.cpp park-unpark.
    //
    // Beware -- Some versions of NPTL embody a flaw where pthread_cond_timedwait() can
    // hang indefinitely.  For instance NPTL 0.60 on 2.4.21-4ELsmp is vulnerable.
    // For specifics regarding the bug see GLIBC BUGID 261237 :
    //    http://www.mail-archive.com/debian-glibc@lists.debian.org/msg10837.html.
    // Briefly, pthread_cond_timedwait() calls with an expiry time that's not in the future
    // will either hang or corrupt the condvar, resulting in subsequent hangs if the condvar
    // is used.  (The simple C test-case provided in the GLIBC bug report manifests the
    // hang).  The JVM is vulernable via sleep(), Object.wait(timo), LockSupport.parkNanos()
    // and monitorenter when we're using 1-0 locking.  All those operations may result in
    // calls to pthread_cond_timedwait().  Using LD_ASSUME_KERNEL to use an older version
    // of libpthread avoids the problem, but isn't practical.
    //
    // Possible remedies:
    //
    // 1.   Establish a minimum relative wait time.  50 to 100 msecs seems to work.
    //      This is palliative and probabilistic, however.  If the thread is preempted
    //      between the call to compute_abstime() and pthread_cond_timedwait(), more
    //      than the minimum period may have passed, and the abstime may be stale (in the
    //      past) resultin in a hang.   Using this technique reduces the odds of a hang
    //      but the JVM is still vulnerable, particularly on heavily loaded systems.
    //
    // 2.   Modify park-unpark to use per-thread (per ParkEvent) pipe-pairs instead
    //      of the usual flag-condvar-mutex idiom.  The write side of the pipe is set
    //      NDELAY. unpark() reduces to write(), park() reduces to read() and park(timo)
    //      reduces to poll()+read().  This works well, but consumes 2 FDs per extant
    //      thread.
    //
    // 3.   Embargo pthread_cond_timedwait() and implement a native "chron" thread
    //      that manages timeouts.  We'd emulate pthread_cond_timedwait() by enqueuing
    //      a timeout request to the chron thread and then blocking via pthread_cond_wait().
    //      This also works well.  In fact it avoids kernel-level scalability impediments
    //      on certain platforms that don't handle lots of active pthread_cond_timedwait()
    //      timers in a graceful fashion.
    //
    // 4.   When the abstime value is in the past it appears that control returns
    //      correctly from pthread_cond_timedwait(), but the condvar is left corrupt.
    //      Subsequent timedwait/wait calls may hang indefinitely.  Given that, we
    //      can avoid the problem by reinitializing the condvar -- by cond_destroy()
    //      followed by cond_init() -- after all calls to pthread_cond_timedwait().
    //      It may be possible to avoid reinitialization by checking the return
    //      value from pthread_cond_timedwait().  In addition to reinitializing the
    //      condvar we must establish the invariant that cond_signal() is only called
    //      within critical sections protected by the adjunct mutex.  This prevents
    //      cond_signal() from "seeing" a condvar that's in the midst of being
    //      reinitialized or that is corrupt.  Sadly, this invariant obviates the
    //      desirable signal-after-unlock optimization that avoids futile context switching.
    //
    //      I'm also concerned that some versions of NTPL might allocate an auxilliary
    //      structure when a condvar is used or initialized.  cond_destroy()  would
    //      release the helper structure.  Our reinitialize-after-timedwait fix
    //      put excessive stress on malloc/free and locks protecting the c-heap.
    //
    // We currently use (4).  See the WorkAroundNTPLTimedWaitHang flag.
    // It may be possible to refine (4) by checking the kernel and NTPL verisons
    // and only enabling the work-around for vulnerable environments.
```

### Solaris 上での実装について
Solaris 版では, 以下の2つのオプションによって
使用するスレッド実行制御用の関数を切り替えられるようになっている.

  * UseLWPSynchronization
  * UsePthreads

具体的には, 以下のように切り替えられる.

  * UseLWPSynchronization が true の場合:
    
    Solaris スレッドの LWP 用の制御関数を使用する.

  * UsePthreads が true の場合:

    pthread の制御関数を使用する.

  * どちらも false の場合:
    
    Solaris スレッドのユーザースレッド用の制御関数を使用する.


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.hpp))
      // initialized to libthread or lwp synchronization primitives depending on UseLWPSychronization
      static int_fnP_mutex_tP _mutex_lock;
      static int_fnP_mutex_tP _mutex_trylock;
      static int_fnP_mutex_tP _mutex_unlock;
      static int_fnP_mutex_tP_i_vP _mutex_init;
      static int_fnP_mutex_tP _mutex_destroy;
```


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.hpp))
      static int_fnP_cond_tP_mutex_tP_timestruc_tP _cond_timedwait;
      static int_fnP_cond_tP_mutex_tP _cond_wait;
      static int_fnP_cond_tP _cond_signal;
      static int_fnP_cond_tP _cond_broadcast;
      static int_fnP_cond_tP_i_vP _cond_init;
      static int_fnP_cond_tP _cond_destroy;
```


```cpp
    ((cite: hotspot/src/os/solaris/vm/os_solaris.cpp))
    void os::Solaris::synchronization_init() {
      if(UseLWPSynchronization) {
        os::Solaris::set_mutex_lock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("_lwp_mutex_lock")));
        os::Solaris::set_mutex_trylock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("_lwp_mutex_trylock")));
        os::Solaris::set_mutex_unlock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("_lwp_mutex_unlock")));
        os::Solaris::set_mutex_init(lwp_mutex_init);
        os::Solaris::set_mutex_destroy(lwp_mutex_destroy);
        os::Solaris::set_mutex_scope(USYNC_THREAD);
    
        os::Solaris::set_cond_timedwait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP_timestruc_tP, resolve_symbol("_lwp_cond_timedwait")));
        os::Solaris::set_cond_wait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP, resolve_symbol("_lwp_cond_wait")));
        os::Solaris::set_cond_signal(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("_lwp_cond_signal")));
        os::Solaris::set_cond_broadcast(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("_lwp_cond_broadcast")));
        os::Solaris::set_cond_init(lwp_cond_init);
        os::Solaris::set_cond_destroy(lwp_cond_destroy);
        os::Solaris::set_cond_scope(USYNC_THREAD);
      }
      else {
        os::Solaris::set_mutex_scope(USYNC_THREAD);
        os::Solaris::set_cond_scope(USYNC_THREAD);
    
        if(UsePthreads) {
          os::Solaris::set_mutex_lock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("pthread_mutex_lock")));
          os::Solaris::set_mutex_trylock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("pthread_mutex_trylock")));
          os::Solaris::set_mutex_unlock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("pthread_mutex_unlock")));
          os::Solaris::set_mutex_init(pthread_mutex_default_init);
          os::Solaris::set_mutex_destroy(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("pthread_mutex_destroy")));
    
          os::Solaris::set_cond_timedwait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP_timestruc_tP, resolve_symbol("pthread_cond_timedwait")));
          os::Solaris::set_cond_wait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP, resolve_symbol("pthread_cond_wait")));
          os::Solaris::set_cond_signal(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("pthread_cond_signal")));
          os::Solaris::set_cond_broadcast(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("pthread_cond_broadcast")));
          os::Solaris::set_cond_init(pthread_cond_default_init);
          os::Solaris::set_cond_destroy(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("pthread_cond_destroy")));
        }
        else {
          os::Solaris::set_mutex_lock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("mutex_lock")));
          os::Solaris::set_mutex_trylock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("mutex_trylock")));
          os::Solaris::set_mutex_unlock(CAST_TO_FN_PTR(int_fnP_mutex_tP, resolve_symbol("mutex_unlock")));
          os::Solaris::set_mutex_init(::mutex_init);
          os::Solaris::set_mutex_destroy(::mutex_destroy);
    
          os::Solaris::set_cond_timedwait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP_timestruc_tP, resolve_symbol("cond_timedwait")));
          os::Solaris::set_cond_wait(CAST_TO_FN_PTR(int_fnP_cond_tP_mutex_tP, resolve_symbol("cond_wait")));
          os::Solaris::set_cond_signal(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("cond_signal")));
          os::Solaris::set_cond_broadcast(CAST_TO_FN_PTR(int_fnP_cond_tP, resolve_symbol("cond_broadcast")));
          os::Solaris::set_cond_init(::cond_init);
          os::Solaris::set_cond_destroy(::cond_destroy);
        }
      }
    }
```


## 処理の流れ (概要)(Execution Flows : Summary)
### ParkEvent::Allocate() の流れ
```
ParkEvent::Allocate()
```

### os::PlatformEvent::park() の流れ
#### Linux の場合
```
os::PlatformEvent::park()
-> pthread_cond_wait()
```

#### Solaris の場合
```
os::PlatformEvent::park()
-> os::Solaris::cond_wait()
   -> _lwp_cond_wait() or pthread_cond_wait() or cond_wait()
```

#### Windows の場合
```
os::PlatformEvent::park()
-> WaitForSingleObject()
```


### os::PlatformEvent::park(jlong millis) の流れ
#### Linux の場合
```
os::PlatformEvent::park(jlong millis)
-> os::Linux::safe_cond_timedwait()
   -> pthread_cond_timedwait()
```

#### Solaris の場合
```
os::PlatformEvent::park(jlong millis)
-> os::Solaris::cond_timedwait()
   -> _lwp_cond_timedwait() or pthread_cond_timedwait() or cond_timedwait()
```

#### Windows の場合
```
os::PlatformEvent::park(jlong millis)
-> WaitForSingleObject()
```


### os::PlatformEvent::unpark() の流れ
#### Linux の場合
```
os::PlatformEvent::unpark()
-> pthread_cond_signal()
```

#### Solaris の場合
```
os::PlatformEvent::unpark()
-> os::Solaris::cond_signal()
   -> _lwp_cond_signal() or pthread_cond_signal() or cond_signal()
```

#### Windows の場合
```
os::PlatformEvent::unpark()
-> SetEvent()
```


### ParkEvent::Release() の流れ
```
ParkEvent::Release()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### ParkEvent::Allocate()
See: [here](no2114n4I.html) for details

### os::PlatformEvent::reset()  (Linux の場合)
See: [here](no2114nxU.html) for details
### os::PlatformEvent::reset()  (Solaris の場合)
See: [here](no211407a.html) for details
### os::PlatformEvent::reset()  (Windows の場合)
See: [here](no2114BGh.html) for details

### os::PlatformEvent::park()  (Linux の場合)
See: [here](no21140CP.html) for details
### os::PlatformEvent::park()  (Solaris の場合)
See: [here](no2114orn.html) for details
### os::Solaris::mutex_lock()
See: [here](no2114boV.html) for details
### os::Solaris::cond_wait()
See: [here](no2114OeP.html) for details
### os::Solaris::mutex_unlock()
See: [here](no2114oyb.html) for details
### os::PlatformEvent::park()  (Windows の場合)
See: [here](no211418h.html) for details

### os::PlatformEvent::park(jlong millis)  (Linux の場合)
See: [here](no2114BNV.html) for details
### os::Linux::safe_cond_timedwait()
See: [here](no2114OXb.html) for details
### os::PlatformEvent::park(jlong millis)  (Solaris の場合)
See: [here](no2114CHo.html) for details
### os::Solaris::cond_timedwait()
See: [here](no2114cb0.html) for details
### os::PlatformEvent::park(jlong millis)  (Windows の場合)
See: [here](no2114PRu.html) for details

### os::PlatformEvent::unpark()  (Linux の場合)
See: [here](no2114bat.html) for details
### os::PlatformEvent::unpark()  (Solaris の場合)
See: [here](no2114o5P.html) for details
### os::Solaris::cond_signal()
See: [here](no21141DW.html) for details
### os::PlatformEvent::unpark()  (Windows の場合)
See: [here](no2114OlD.html) for details

### ParkEvent::Release()
See: [here](no2114auC.html) for details







