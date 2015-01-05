---
layout: default
title: プリミティブなロック／アンロック処理 (java.util.concurrent.locks.LockSupport.park(), java.util.concurrent.locks.LockSupport.unpark() の処理)  
---
[Up](no1IkYYOWe.html) [Top](../index.html)

#### プリミティブなロック／アンロック処理 (java.util.concurrent.locks.LockSupport.park(), java.util.concurrent.locks.LockSupport.unpark() の処理)  

--- 
## 概要(Summary)
sun.misc.Unsafe クラスを経由して, 
最終的には Parker オブジェクトの Parker::park(), Parker::unpark() が呼び出される.


## 内部の構造
### Linux 版および Solaris 版の実装について
現状では, mutex, condvar 及び count という変数 (実際のコード中では _counter) を用いている.

* park() 時には, count の値を 1 から 0 にデクリメントする. 
  ただし count が既に 0 になっていれば condvar wait する.

* unpark() 時には, count の値を 1 にセットし, condvar signal する.

condvar 上では 1つのスレッドしか待機できないので, 競合が起きるのは park() と unpark() がぶつかった場合.
このため競合が起きた場合には待機の必要は無い.
また spurious wakeup はしてもかまわない.


```cpp
    ((cite: hotspot/src/os/linux/vm/os_linux.cpp))
    /*
     * The solaris and linux implementations of park/unpark are fairly
     * conservative for now, but can be improved. They currently use a
     * mutex/condvar pair, plus a a count.
     * Park decrements count if > 0, else does a condvar wait.  Unpark
     * sets count to 1 and signals condvar.  Only one thread ever waits
     * on the condvar. Contention seen when trying to park implies that someone
     * is unparking you, so don't wait. And spurious returns are fine, so there
     * is no need to track notifications.
     */
```

### sun.misc.Unsafe クラスのネイティブメソッドについて
sun.misc.Unsafe の native メソッドは
JVM_RegisterUnsafeMethods() で登録されている.

sun.misc.Unsafe.park() メソッドや sun.misc.Unsafe.unpark() メソッドの対応関係については,
JDK 1.6 以降なら methods という配列に, JDK 1.5 なら methods_15 という配列に納められている.


## 処理の流れ (概要)(Execution Flows : Summary)
### java.util.concurrent.locks.LockSupport.park() 処理の流れ
<div class="flow-abst"><pre>
java.util.concurrent.locks.LockSupport.park()
-&gt; Unsafe_Park() (= sun.misc.Unsafe.park())
   -&gt; Parker::park()
      -&gt; OS によって処理が異なる.
         * Linux の場合
           -&gt; pthread_mutex_trylock()
           -&gt; pthread_cond_wait() または os::Linux::safe_cond_timedwait()
           -&gt; pthread_mutex_unlock()
         * Solaris の場合
           -&gt; os::Solaris::mutex_trylock()
           -&gt; os::Solaris::cond_wait() または os::Solaris::cond_timedwait()
           -&gt; os::Solaris::mutex_unlock()
         * Windows の場合
           -&gt; WaitForSingleObject()
           -&gt; ResetEvent()
</pre></div>

### java.util.concurrent.locks.LockSupport.unpark() 処理の流れ
<div class="flow-abst"><pre>
java.util.concurrent.locks.LockSupport.unpark()
-&gt; Unsafe_Unpark() (= sun.misc.Unsafe.unpark())
   -&gt; Parker::unpark()
      -&gt; OS によって処理が異なる.
         * Linux の場合
           -&gt; pthread_mutex_lock()
           -&gt; pthread_mutex_unlock()
           -&gt; pthread_cond_signal()
         * Solaris の場合
           -&gt; os::Solaris::mutex_lock()
           -&gt; os::Solaris::mutex_unlock()
           -&gt; os::Solaris::cond_signal()
         * Windows の場合
           -&gt; SetEvent()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### java.util.concurrent.locks.LockSupport.park()
See: [here](no2114nqg.html) for details
### Unsafe_Park()
See: [here](no2114B_s.html) for details
### Parker::park() (Linux の場合)
See: [here](no9662Q2N.html) for details
### Parker::park() (Solaris の場合)
See: [here](no9662Rps.html) for details
### Parker::park() (Windows の場合)
See: [here](no9662dAU.html) for details

### java.util.concurrent.locks.LockSupport.unpark()
See: [here](no211400m.html) for details
### Unsafe_Unpark()
See: [here](no2114OJz.html) for details
### Parker::unpark() (Linux の場合)
See: [here](no96623Ug.html) for details
### Parker::unpark() (Solaris の場合)
See: [here](no2114bvJ.html) for details
### Parker::unpark() (Windows の場合)
See: [here](no9662qKa.html) for details






