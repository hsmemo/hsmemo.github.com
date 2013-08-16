---
layout: default
title: VM_Operation とその(基本的な)サブクラス (VM_Operation, VM_ThreadStop, VM_ForceSafepoint, VM_ForceAsyncSafepoint, VM_Deoptimize, VM_DeoptimizeFrame, VM_HandleFullCodeCache, VM_DeoptimizeAll, VM_ZombieAll, VM_UnlinkSymbols, VM_Verify, VM_PrintThreads, VM_PrintJNI, VM_FindDeadlocks, VM_ThreadDump, VM_Exit)
---
[Top](../index.html)

#### VM_Operation とその(基本的な)サブクラス (VM_Operation, VM_ThreadStop, VM_ForceSafepoint, VM_ForceAsyncSafepoint, VM_Deoptimize, VM_DeoptimizeFrame, VM_HandleFullCodeCache, VM_DeoptimizeAll, VM_ZombieAll, VM_UnlinkSymbols, VM_Verify, VM_PrintThreads, VM_PrintJNI, VM_FindDeadlocks, VM_ThreadDump, VM_Exit)

これらは, VM Operation 処理のためのクラス
(See: [here](no2480qPC.html) and [here](no2480eqy.html) for details).


### クラス一覧(class list)

  * [VM_Operation](#novapJesDb)
  * [VM_ThreadStop](#noXdTVViuw)
  * [VM_ForceSafepoint](#no9SuranI5)
  * [VM_ForceAsyncSafepoint](#noCrcZMHoh)
  * [VM_Deoptimize](#noXGrRTis-)
  * [VM_DeoptimizeFrame](#noUT9mS0T5)
  * [VM_HandleFullCodeCache](#noe8g-Gsh2)
  * [VM_DeoptimizeAll](#noew0yG4CA)
  * [VM_ZombieAll](#noZdPSEVWt)
  * [VM_UnlinkSymbols](#nobcG7IDZ8)
  * [VM_Verify](#no7wpCQf7b)
  * [VM_PrintThreads](#noSivciXQf)
  * [VM_PrintJNI](#no8Z5CLS6E)
  * [VM_FindDeadlocks](#no9l616md3)
  * [VM_ThreadDump](#nol3HpsJaP)
  * [VM_Exit](#nocsVimgo7)


---
## <a name="novapJesDb" id="novapJesDb">VM_Operation</a>

### 概要(Summary)
全ての VM Operation クラスの基底クラス.

なお, このクラス自体は abstract class であり, 実際に使われるのはサブクラス.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_Operation: public CHeapObj {
```

### 使われ方(Usage)
使用する際には, doit() メソッドをオーバーライドしたサブクラスを作ればいい.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
      virtual void doit()                            = 0;
```




### 詳細(Details)
See: [here](../doxygen/classVM__Operation.html) for details

---
## <a name="noXdTVViuw" id="noXdTVViuw">VM_ThreadStop</a>

### 概要(Summary)
java.lang.Thread の停止処理用 (java.lang.Thread.stop() 用) の補助クラス(VM_Operation クラス)
(See: [here](no2114VUD.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_ThreadStop: public VM_Operation {
```

### 使われ方(Usage)
Thread::send_async_exception() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
* java.lang.Thread.stop() の処理

  (略) (See: [here](no2114VUD.html) for details)
  -> JVM_StopThread()
     -> Thread::send_async_exception()

* JVMTI の StopThread() の処理

  (略) (See: [here](noeOxh-Gl_.html) for details)
  -> JvmtiEnv::StopThread()
     -> Thread::send_async_exception()

* (?? #TODO)
  
  -> InlineCacheBuffer::new_ic_stub()
     -> Thread::send_async_exception()
```




### 詳細(Details)
See: [here](../doxygen/classVM__ThreadStop.html) for details

---
## <a name="no9SuranI5" id="no9SuranI5">VM_ForceSafepoint</a>

### 概要(Summary)
ダミーの VM_Operation クラス. 処理は何も行わない.

Safepoint を発生させることで間接的に NMethodSweeper::scan_stacks() 等を実行したい, という時に使われる
(See: SafepointSynchronize::do_cleanup_tasks()).

```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    // dummy vm op, evaluated just to force a safepoint
    class VM_ForceSafepoint: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* ClassLoader::compile_the_world_in()
* InlineCacheBuffer::new_ic_stub()
* JvmtiEnv::SuspendThreadList()
* JavaThread::java_suspend()

### 内部構造(Internal structure)
(このクラス自体は何も処理はしない)

```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
      void doit()         {}
```




### 詳細(Details)
See: [here](../doxygen/classVM__ForceSafepoint.html) for details

---
## <a name="noCrcZMHoh" id="noCrcZMHoh">VM_ForceAsyncSafepoint</a>

### 概要(Summary)
ダミーの VM_Operation クラス. 処理は何も行わない.

Safepoint を発生させることで間接的に ObjectSynchronizer::deflate_idle_monitors() 等を実行したい, という時に使われる
(See: SafepointSynchronize::do_cleanup_tasks()).

```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    // dummy vm op, evaluated just to force a safepoint
    class VM_ForceAsyncSafepoint: public VM_Operation {
```

### 使われ方(Usage)
InduceScavenge() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
ObjectSynchronizer::inflate()
-> ObjectSynchronizer::omAlloc()
   -> InduceScavenge()
```

### 内部構造(Internal structure)
(このクラス自体は何も処理はしない)

```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
      void doit()              {}
```




### 詳細(Details)
See: [here](../doxygen/classVM__ForceAsyncSafepoint.html) for details

---
## <a name="noXGrRTis-" id="noXGrRTis-">VM_Deoptimize</a>

### 概要(Summary)
JIT 生成されたコード(nmethod)に対する脱最適化処理(Deoptimization処理)で使用される補助クラス(VM_Operation クラス) 
(See: [here](no3420xYb.html) for details).

CodeCache::mark_for_deoptimization() でマークを付けられた全ての nmethod 
(及びそれに依存している nmethod) に対して脱最適化を行う.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_Deoptimize: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* (ダイナミックロード等で) クラス階層に変化があった場合

  -> SystemDictionary::add_to_hierarchy()
     -> Universe::flush_dependents_on()

* JVMTI のイベント通知処理のために interp_only_mode になる際 (See: [here](no3059eFS.html) and [here](no2935C7Z.html) for details)

  VM_EnterInterpOnlyMode::doit()
```




### 詳細(Details)
See: [here](../doxygen/classVM__Deoptimize.html) for details

---
## <a name="noUT9mS0T5" id="noUT9mS0T5">VM_DeoptimizeFrame</a>

### 概要(Summary)
JIT 生成されたコード(nmethod)に対する脱最適化処理(Deoptimization処理)で使用される補助クラス(VM_Operation クラス) 
(See: [here](no3420xYb.html) for details).

指定されたスレッドの指定されたスタックフレームを脱最適化する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    // Deopt helper that can deoptimize frames in threads other than the
    // current thread.  Only used through Deoptimization::deoptimize_frame.
    class VM_DeoptimizeFrame: public VM_Operation {
```

### 使われ方(Usage)
Deoptimization::deoptimize_frame() 内で(のみ)使用されている.
この関数は, 現在は以下のパスで(のみ)呼び出されている.

```
* C1 の ...#TODO 処理

  Runtime1::patch_code()
  -> Deoptimization::deoptimize_frame()

* C1 の ...#TODO 処理

  deopt_caller()
  -> Deoptimization::deoptimize_frame()

* C1 の ...#TODO 処理

  Runtime1::counter_overflow()
  -> Deoptimization::deoptimize_frame()

* C1 の ...#TODO 処理

  exception_handler_for_pc_helper()
  -> Deoptimization::deoptimize_frame()


* C2 の ...#TODO 処理

  OptoRuntime::deoptimize_caller_frame()
  -> Deoptimization::deoptimize_frame()

* C2 生成コードでの Safepoint 処理中

  (略) (See: [here](no7882Okb.html) for details)
  -> SafepointSynchronize::handle_polling_page_exception()
     -> ThreadSafepointState::handle_polling_page_exception()
        -> Deoptimization::deoptimize_frame()


* JVMTI の PopFrame() 関数の処理

  (略) (See: [here](no2935cDo.html) for details)
  -> JvmtiEnv::PopFrame()
     -> Deoptimization::deoptimize_frame()

* JVMTI の ForceEarlyReturn*() 関数の処理

  (略) (See: [here](no3059azN.html) for details)
  -> JvmtiEnvBase::force_early_return()
     -> JvmtiEnvBase::check_top_frame()
        -> Deoptimization::deoptimize_frame()

* JVMTI の GetLocal*() 及び SetLocal*() 関数の処理

  (略) (See: [here](no2935GIU.html) for details)
  -> VM_GetOrSetLocal::doit()
     -> Deoptimization::deoptimize_frame()
```





### 詳細(Details)
See: [here](../doxygen/classVM__DeoptimizeFrame.html) for details

---
## <a name="noe8g-Gsh2" id="noe8g-Gsh2">VM_HandleFullCodeCache</a>

### 概要(Summary)
NMethodSweeper クラス内で使用される補助クラス(VM_Operation クラス).

CodeCache が一杯になった際に実行され, 古い nmethod を CodeCache 内から廃棄する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_HandleFullCodeCache: public VM_Operation {
```

### 使われ方(Usage)
NMethodSweeper::handle_full_code_cache() 内で(のみ)使用されている.

### 内部構造(Internal structure)
NMethodSweeper::speculative_disconnect_nmethods() を呼び出すだけ.

#### 参考(for your information): VM_HandleFullCodeCache::doit()

```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.cpp))
    void VM_HandleFullCodeCache::doit() {
      NMethodSweeper::speculative_disconnect_nmethods(_is_full);
    }
```




### 詳細(Details)
See: [here](../doxygen/classVM__HandleFullCodeCache.html) for details

---
## <a name="noew0yG4CA" id="noew0yG4CA">VM_DeoptimizeAll</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない)
(なお, このクラスは (デバッグ時であることに加えて) 
 DeoptimizeALot オプションまたは DeoptimizeRandom オプションが指定されている場合にしか働かない).

VMEntryWrapper クラス内で使用される補助クラス(VM_Operation クラス).

全てのスタックフレーム(またはランダムで選んだスタックフレーム)を脱最適化する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_DeoptimizeAll: public VM_Operation {
```

### 使われ方(Usage)
InterfaceSupport::deoptimizeAll() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
VMEntryWrapper()::~VMEntryWrapper()
-> InterfaceSupport::deoptimizeAll()
```

なお JRT_ENTRY や IRT_ENTRY の度に実行すると重いので, 
InterfaceSupport::deoptimizeAll() では以下のタイミングで VM_DeoptimizeAll を実行することにしている模様

  * DeoptimizeALot の場合は, DeoptimizeALotInterval 回に 1回実行する
  * DeoptimizeRandom の場合は, ランダムに実行するかしないかを決める

#### 参考(for your information): InterfaceSupport::deoptimizeAll()
See: [here](no289166XY.html) for details
### 内部構造(Internal structure)
内部では以下の処理を行う.

  * DeoptimizeALot の場合は, 全てのスレッドに JavaThread::deoptimize() をかけて, 全部のフレームを deoptimize する.
  * DeoptimizeRandom の場合は, ランダムに選んだスレッドのランダムに選んだフレームを deoptimize する.

なお, JavaThread::deoptimize() は, 
全部のフレームに対して Deoptimization::deoptimize() を適用する関数
(DeoptimizeOnlyAt コマンドラインオプションで脱最適化の範囲を絞ることも出来る).

#### 参考(for your information): VM_DeoptimizeAll::doit()
See: [here](no28916Hie.html) for details
#### 参考(for your information): JavaThread::deoptimize()
See: [here](no28916h2q.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__DeoptimizeAll.html) for details

---
## <a name="noZdPSEVWt" id="noZdPSEVWt">VM_ZombieAll</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifndef PRODUCT 時にしか定義されない).
(なお, このクラスは (デバッグ時であることに加えて) 
 ZombieALot オプションが指定されている場合にしか働かない).

VMEntryWrapper クラス内で使用される補助クラス(VM_Operation クラス).

呼び出したスレッドのスタックフレーム内を調べ, 
JIT 生成されたメソッドがあれば対応する JIT 生成コード(nmethod)を破棄する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    #ifndef PRODUCT
```


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_ZombieAll: public VM_Operation {
```

### 使われ方(Usage)
InterfaceSupport::zombieAll() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
VMEntryWrapper()::~VMEntryWrapper()
-> InterfaceSupport::zombieAll()
```

なお JRT_ENTRY や IRT_ENTRY の度に実行すると重いので, 
InterfaceSupport::zombieAll() では ZombieALotInterval 回に 1回実行することにしている模様.

#### 参考(for your information): InterfaceSupport::zombieAll()
See: [here](no28916gDM.html) for details
### 内部構造(Internal structure)
JavaThread::make_zombies() を呼び出すだけ.

なお, JavaThread::make_zombies() は, 
呼び出したスレッドのスタックフレーム内に JIT 生成されたメソッドがあれば
nmthod::make_not_entrant() で強制的に破棄させる関数.

#### 参考(for your information): VM_ZombieAll::doit()
See: [here](no28916hv2.html) for details
#### 参考(for your information): JavaThread::make_zombies()
See: [here](no28916T5F.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__ZombieAll.html) for details

---
## <a name="nobcG7IDZ8" id="nobcG7IDZ8">VM_UnlinkSymbols</a>

### 概要(Summary)
デバッグ用(開発時用)のクラス (#ifdef ASSERT 時にしか使用されない)
(なお, このクラスは (デバッグ時であることに加えて) 
 UnlinkSymbolsALot オプションが指定されている場合にしか働かない).

VMEntryWrapper クラス内で使用される補助クラス(VM_Operation クラス).

SymbolTable 中の参照されていないシンボル(dead symbol)を全て破棄させる.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_UnlinkSymbols: public VM_Operation {
```

### 使われ方(Usage)
InterfaceSupport::unlinkSymbols() 内で(のみ)使用されている.
そして, この関数は現在は以下のパスで(のみ)呼び出されている.

```
VMEntryWrapper()::~VMEntryWrapper()
-> InterfaceSupport::unlinkSymbols()
```


```
    ((cite: hotspot/src/share/vm/runtime/interfaceSupport.cpp))
    void InterfaceSupport::unlinkSymbols() {
      VM_UnlinkSymbols op;
      VMThread::execute(&op);
    }
```

### 内部構造(Internal structure)
SymbolTable::unlink() を呼び出して, SymbolTable 中の参照されていないシンボル(dead symbol)を全て破棄させるだけ.

#### 参考(for your information): VM_UnlinkSymbols::doit()
See: [here](no289166Qk.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__UnlinkSymbols.html) for details

---
## <a name="no7wpCQf7b" id="no7wpCQf7b">VM_Verify</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

デバッグ用(開発時用)のクラス.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_Verify: public VM_Operation {
```

### 内部構造(Internal structure)
Universe::verify() を呼び出すだけ.

#### 参考(for your information): VM_Verify::doit()
See: [here](no28916g8X.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__Verify.html) for details

---
## <a name="noSivciXQf" id="noSivciXQf">VM_PrintThreads</a>

### 概要(Summary)
保守運用機能のためのクラス (Dynamic Attach 機能及びスレッドダンプ機能用のクラス).

HotSpot 内で稼働中の全スレッドについて, 
その実行状態(RUNNABLE, WAITING 等)とスタックトレースを出力する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_PrintThreads: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* スレッドダンプ機能
  
  (略) (See: [here](no28916GhX.html) for details)
  -> signal_thread_entry()

* Attach API の threaddump コマンド
  
  (略) (See: [here](no3026gMG.html) for details)
  -> attach_listener_thread_entry()
     -> thread_dump()

* ?? 
  
  (略) (?? 使われていない #TODO)
  -> JVM_DumpAllStacks()
```
     
### 内部構造(Internal structure)
Threads::print_on() を呼び出すだけ.

#### 参考(for your information): VM_PrintThreads::doit()
See: [here](no28916t_p.html) for details
#### 参考(for your information): Threads::print_on()
See: [here](no28916vPg.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__PrintThreads.html) for details

---
## <a name="no8Z5CLS6E" id="no8Z5CLS6E">VM_PrintJNI</a>

### 概要(Summary)
保守運用機能のためのクラス (Dynamic Attach 機能及びスレッドダンプ機能用のクラス).

現在の JNI Global References の情報を出力する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_PrintJNI: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* スレッドダンプ機能
  
  (略) (See: [here](no28916GhX.html) for details)
  -> signal_thread_entry()

* Attach API の threaddump コマンド
  
  (略) (See: [here](no3026gMG.html) for details)
  -> attach_listener_thread_entry()
     -> thread_dump()
```

### 内部構造(Internal structure)
JNIHandles::print_on() を呼び出すだけ.

#### 参考(for your information): VM_PrintJNI::doit()
See: [here](no289166Jw.html) for details



### 詳細(Details)
See: [here](../doxygen/classVM__PrintJNI.html) for details

---
## <a name="no9l616md3" id="no9l616md3">VM_FindDeadlocks</a>

### 概要(Summary)
保守運用機能のためのクラス (関連する JMM 機能, Dynamic Attach 機能及びスレッドダンプ機能用のクラス).

デッドロックしているスレッド群がいないか調べ, その結果を表示する.


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_FindDeadlocks: public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* スレッドダンプ機能
  
  (略) (See: [here](no28916GhX.html) for details)
  -> signal_thread_entry()

* Attach API の threaddump コマンド
  
  (略) (See: [here](no3026gMG.html) for details)
  -> attach_listener_thread_entry()
     -> thread_dump()

* JMM のデッドロック検出処理 (java.lang.management.ThreadMXBean.findDeadlockedThreads() および java.lang.management.ThreadMXBean.findMonitorDeadlockedThreads() の処理)

  (略) (See: [here](no2114hUk.html) for details)
  -> jmm_FindDeadlockedThreads()
     -> find_deadlocks()

  (略) (See: [here](no2114hUk.html) for details)
  -> jmm_FindMonitorDeadlockedThreads()
     -> find_deadlocks()
```




### 詳細(Details)
See: [here](../doxygen/classVM__FindDeadlocks.html) for details

---
## <a name="nol3HpsJaP" id="nol3HpsJaP">VM_ThreadDump</a>

### 概要(Summary)
java.lang.Thread クラスの機能, 及び JMM の機能を実現するための補助クラス(VM_Operation クラス).

スレッドの情報を取得する 
(スタックトレース情報, 現在ロックしているシンクロナイザの一覧, 現在ロックしているオブジェクトモニターの一覧, etc).


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_ThreadDump : public VM_Operation {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* java.lang.Thread のスタックフレーム取得処理 (java.lang.Thread.getStackTrace(), java.lang.Thread.getAllStackTraces() の処理)

  (略) (See: [here](no2114ieJ.html) for details)
  -> ThreadService::dump_stack_traces()

* JMM の java.lang.management.ThreadInfo オブジェクト取得処理 (java.lang.management.ThreadMXBean.getThreadInfo() および java.lang.management.ThreadMXBean.dumpAllThreads() の処理)

  (略) (See: [here](no2114sqE.html) for details)
  -> jmm_GetThreadInfo()
     -> do_thread_dump()

  (略) (See: [here](no2114sqE.html) for details)
  -> jmm_DumpThreads()

  (略) (See: [here](no2114sqE.html) for details)
  -> jmm_DumpThreads()
     -> do_thread_dump()
```




### 詳細(Details)
See: [here](../doxygen/classVM__ThreadDump.html) for details

---
## <a name="nocsVimgo7" id="nocsVimgo7">VM_Exit</a>

### 概要(Summary)
HotSpot の終了処理で使用される補助クラス(VM_Operation クラス).

終了時の後始末を行う (See: [here](no28916GoL.html) for details).


```
    ((cite: hotspot/src/share/vm/runtime/vm_operations.hpp))
    class VM_Exit: public VM_Operation {
```

### 使われ方(Usage)
#### 使用箇所(where its instances are used)
vm_exit() 内で(のみ)使用されている.




### 詳細(Details)
See: [here](../doxygen/classVM__Exit.html) for details

---
