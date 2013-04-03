---
layout: default
title: ExtendedPC クラス 
---
[Top](../index.html)

#### ExtendedPC クラス 



---
## <a name="no7o1K8pWv" id="no7o1K8pWv">ExtendedPC</a>

### 概要(Summary)
os クラス用の補助クラス.

CPU の PC (program counter) の値を表すためのクラス.
より具体的に言うと, 
os::get_thread_pc() と 
os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp) の返値を表すためのクラス.


(中身は単に address 型の値を1つ格納しているだけなので,
 address 型の typedef でも問題ないような気がする.
 ただし,「単なる address ではなく PC の値を表すアドレス」だと型レベルで明示することで
 (ByteSize クラスや WordSize クラスのように) バグを防ぐ効果があるのかもしれない (#TODO))


```
    ((cite: hotspot/src/share/vm/runtime/extendedPC.hpp))
    // An ExtendedPC contains the _pc from a signal handler in a platform
    // independent way.
    
    class ExtendedPC VALUE_OBJ_CLASS_SPEC {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
1. os::get_thread_pc() や os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp) で, 
   PC の値を ExcendedPC オブジェクトとして取得する.

2. ExcendedPC.pc() で, PC のアドレスを示す address 型の値が取得できる.

#### 生成箇所(where its instances are created)
以下の箇所で(のみ)生成されている
(ValueObj クラスなので「生成」というのは少し違和感があるが).

* os::get_thread_pc()
  
  (なお Solaris の場合は, 補助クラスである GetThreadPC_Callback によって生成される)

* os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)

そして, これらの関数は現在は以下のパスで(のみ)呼び出されている.

```
* FlatProfiler の処理

  FlatProfilerTask::task()
  -> FlatProfiler::record_vm_tick()
     -> os::get_thread_pc()

* Forte との連携処理
  
  (Linux sparc の場合)
  AsyncGetCallTrace()
  -> JavaThread::pd_get_top_frame_for_signal_handler()
     -> os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)

  (Linux sparc の場合)
  ?? 使われていない
  -> os::Linux::fetch_frame_from_ucontext()
     -> os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)

  (Linux x86 の場合)
  AsyncGetCallTrace()
  -> JavaThread::pd_get_top_frame_for_signal_handler()
     -> os::Linux::fetch_frame_from_ucontext()
        -> os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)

  (Solaris sparc の場合) (Solaris x86 の場合)
  AsyncGetCallTrace()
  -> JavaThread::pd_get_top_frame_for_signal_handler()
     -> os::Solaris::fetch_frame_from_ucontext()
        -> os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)

* VMError の処理 (なお Linux zero の場合は使用されていない模様)

  (Linux sparc の場合) (Linux x86 の場合) (Solaris sparc の場合) (Solaris x86 の場合) (Windows x86 の場合)
  VMError::report()
  -> os::fetch_frame_from_context(void* ucVoid)
     -> os::fetch_frame_from_context(void* ucVoid, intptr_t** sp, intptr_t** fp)
```
  
### 内部構造(Internal structure)
内部には以下のフィールド(のみ)を含む
(そして, メソッドはこのフィールドへの getter メソッド(アクセサメソッド)のみ).


```
    ((cite: hotspot/src/share/vm/runtime/extendedPC.hpp))
      address _pc;
```




### 詳細(Details)
See: [here](../doxygen/classExtendedPC.html) for details

---
