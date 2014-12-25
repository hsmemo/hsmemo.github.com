---
layout: default
title: ThreadLocalStorage クラス 
---
[Top](../index.html)

#### ThreadLocalStorage クラス 



---
## <a name="noUbeWeNFN" id="noUbeWeNFN">ThreadLocalStorage</a>

### 概要(Summary)
各スレッドの TLS (Thread Local Storage) にアクセスするためのユーティリティ・クラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

ただし現状では, TLS はネイティブスレッドと Thread オブジェクトの対応付けのためにしか使用されていない.
より具体的に言うと, 各ネイティブスレッドの TLS にはそのスレッドに対応する Thread オブジェクトだけが格納されている.

(このため ThreadLocalStorage クラスは, 
実質上は ThreadLocalStorage::set_thread() して 
ThreadLocalStorage::thread() (もしくは ThreadLocalStorage::get_thread_slow()) するためだけのクラス).


```cpp
    ((cite: hotspot/src/share/vm/runtime/threadLocalStorage.hpp))
    // Interface for thread local storage
```

```cpp
    ((cite: hotspot/src/share/vm/runtime/threadLocalStorage.hpp))
    class ThreadLocalStorage : AllStatic {
```

### 使われ方(Usage)
HotSpot 内で使用されるスレッドは, 初期化中に ThreadLocalStorage::set_thread() を呼び出している
(これにより, ネイティブスレッドと HotSpot の Thread オブジェクトを対応付けが TLS に記録される).

```
スレッドの初期化処理 (See: [here](no2114J7x.html), [here](no7882jgS.html), [here](no2935qaz.html) and [here](no2935d4w.html) for details, (no24805iK)) (他にもある #TODO)
-> Thread::initialize_thread_local_storage()
   -> ThreadLocalStorage::set_thread()
      -> ThreadLocalStorage::pd_set_thread()
         -> (各 OS 毎の TLS 操作処理)
```

また, HotSpot 内で生成していないスレッドの場合も, 
JNI 関数でアタッチされると ThreadLocalStorage::set_thread() で対応付けを行う.

```
(略) (See: [here](noxegGjntv.html) for details)
-> attach_current_thread()
   -> Thread::initialize_thread_local_storage()
      -> (同上)
```

逆に TLS の値が NULL であれば, HotSpot と無関係なスレッド 
(= HotSpot 内で生成されておらず HotSpot に Attach もしていないスレッド) だと分かる.

TLS 内に記録した情報は, 
例えば「JNI 関数内でカレントスレッドに対応する JavaThread を取得して何か処理を行う」といった場面で用いられる
(またスレッドの種別判定などにも用いられている模様 (#TODO)).

### 内部構造(Internal structure)
いくつかのプラットフォーム上では, OS が提供する TLS 機能を使うだけでなく, 処理が高速になるように工夫している.

#### Linux x86-32 版の内部実装
単に pthread_setspecific(), pthread_getspecific() を使うだけではなく,
各スレッドのスタックポインタ(ESP)からテーブル引きで Thread オブジェクトを取得できるようにしている.

やり方は簡単で,単に「メモリ空間長/ページサイズ」分だけの要素数を持った配列を作り 
(1要素が仮想アドレス内の1ページに対応), 
各ネイティブスレッドのスタック領域 (stack base から stack top まで) 
に該当する範囲に対応する Thread オブジェクトを書き込んでおく, というもの
(この配列を ESP の値で引けば対応する Thread オブジェクトが得られる).

(コメントによると, 
 demand paging とはいえこのテーブルは最大 4MB メモリを食うので, 
 問題があれば不要になった部分を madvise() で解放した方がいいかも, とのこと)
(ちなみに 64bit 版でやろうとするとスタック領域を 2MB page にしても最大 512MB. 流石に無理か...)


```cpp
    ((cite: hotspot/src/os_cpu/linux_x86/vm/threadLS_linux_x86.cpp))
    // Map stack pointer (%esp) to thread pointer for faster TLS access
    //
    // Here we use a flat table for better performance. Getting current thread
    // is down to one memory access (read _sp_map[%esp>>12]) in generated code
    // and two in runtime code (-fPIC code needs an extra load for _sp_map).
    //
    // This code assumes stack page is not shared by different threads. It works
    // in 32-bit VM when page size is 4K (or a multiple of 4K, if that matters).
    //
    // Notice that _sp_map is allocated in the bss segment, which is ZFOD
    // (zero-fill-on-demand). While it reserves 4M address space upfront,
    // actual memory pages are committed on demand.
    //
    // If an application creates and destroys a lot of threads, usually the
    // stack space freed by a thread will soon get reused by new thread
    // (this is especially true in NPTL or LinuxThreads in fixed-stack mode).
    // No memory page in _sp_map is wasted.
    //
    // However, it's still possible that we might end up populating &
    // committing a large fraction of the 4M table over time, but the actual
    // amount of live data in the table could be quite small. The max wastage
    // is less than 4M bytes. If it becomes an issue, we could use madvise()
    // with MADV_DONTNEED to reclaim unused (i.e. all-zero) pages in _sp_map.
    // MADV_DONTNEED on Linux keeps the virtual memory mapping, but zaps the
    // physical memory page (i.e. similar to MADV_FREE on Solaris).
    
    #ifndef AMD64
    Thread* ThreadLocalStorage::_sp_map[1UL << (SP_BITLENGTH - PAGE_SHIFT)];
    #endif // !AMD64
```

#### Solaris x86 版の内部実装
単に thr_setspecific(), thr_getspecific() を使うだけではなく,
セグメントレジスタを使って少し最適化している模様 
(See: ThreadLocalStorage::set_thread_in_slot()) (#TODO).

#### 参考(for your information): Thread::initialize_thread_local_storage()
See: [here](no17119iiR.html) for details
#### 参考(for your information): ThreadLocalStorage::set_thread()
See: [here](no30596Wi.html) for details
#### 参考(for your information): ThreadLocalStorage::pd_set_thread() (Solaris の場合)
See: [here](no1711982d.html) for details
#### 参考(for your information): os::thread_local_storage_at_put() (Solaris の場合)
See: [here](no17119JBk.html) for details
#### 参考(for your information): ThreadLocalStorage::set_thread_in_slot() (Solaris x86 の場合)
See: [here](no17119jVw.html) for details
#### 参考(for your information): MacroAssembler::get_thread() (Solaris x86 の場合)
See: [here](no17119WLq.html) for details
#### 参考(for your information): ThreadLocalStorage::pd_set_thread() (Linux x86 の場合)
See: [here](no17119wf2.html) for details
#### 参考(for your information): os::thread_local_storage_at_put() (Linux の場合)
See: [here](no17119ipF.html) for details
#### 参考(for your information): ThreadLocalStorage::pd_set_thread() (Windows x86 の場合)
See: [here](no3059Lzs.html) for details
#### 参考(for your information): os::thread_local_storage_at_put() (Windows x86 の場合)
See: [here](no3059Y9y.html) for details



### 詳細(Details)
See: [here](../doxygen/classThreadLocalStorage.html) for details

---
