---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/os/ 以下 ： linux/
---
[Up](nofy_wtXm1.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/os/ 以下 ： linux/

--- 

File Name                                                         | Description
----------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/os/linux/vm/attachListener_linux.cpp         	  |  AttachListener クラスのプラットフォーム依存なメソッドの定義, 及びその補助クラスの定義 ([LinuxAttachListener, LinuxAttachOperation, ArgumentIterator](norfP3Zc-E.html))
hotspot/src/os/linux/vm/c1_globals_linux.hpp             	  |  空ファイル(※1) (See: [here](no7882y_Y.html) for details)
hotspot/src/os/linux/vm/c2_globals_linux.hpp             	  |  空ファイル(※2) (See: [here](no7882y_Y.html) for details)
hotspot/src/os/linux/vm/chaitin_linux.cpp                	  |  PhaseRegAlloc クラスのプラットフォーム依存なメソッドの定義(PhaseRegAlloc::pd_preallocate_hook(), PhaseRegAlloc::pd_postallocate_verify_hook()) (※3)
hotspot/src/os/linux/vm/decoder_linux.cpp                	  |  Decoder クラスのプラットフォーム依存なメソッドの定義(Decoder::demangle()) (※4)
hotspot/src/os/linux/vm/dtraceJSDT_linux.cpp             	  |  DTraceJSDT クラスのプラットフォーム依存なメソッドの定義(DTraceJSDT::pd_activate(), DTraceJSDT::pd_dispose(), DTraceJSDT::pd_is_supported()) (※5)
hotspot/src/os/linux/vm/globals_linux.hpp                	  |  Linux 独自の JVM のコマンドラインオプションの定義 (あるいはデフォルト値の変更) (See: [here](no7882y_Y.html) for details)
hotspot/src/os/linux/vm/interfaceSupport_linux.hpp       	  |  InterfaceSupport クラス用の関数の定義 (serialize_memory())
hotspot/src/os/linux/vm/jsig.c                           	  |  Signal Chaining 機能用のファイル. シグナルハンドラ設定用の関数(sigaction(), signal(), sigset())を上書きするために同名の独自関数を定義している. (※6)
hotspot/src/os/linux/vm/jvm_linux.cpp                    	  |  sun.misc.Signal クラス用の補助関数(JVM_RegisterSignal(), JVM_RaiseSignal(), JVM_FindSignal()), 及び os::exception_name() 用の補助関数(signal_name()) の定義
hotspot/src/os/linux/vm/jvm_linux.h                      	  |  JNI や CJVM 等に関係したプラットフォーム依存な定数の定義 (JNI_ONLOAD_SYMBOLS, JVM_ONLOAD_SYMBOLS, AGENT_ONLOAD_SYMBOLS, JNI_LIB_PREFIX, JNI_LIB_SUFFIX, etc)
hotspot/src/os/linux/vm/mutex_linux.cpp                  	  |  空ファイル(※7)
hotspot/src/os/linux/vm/mutex_linux.inline.hpp           	  |  同上
hotspot/src/os/linux/vm/osThread_linux.cpp               	  |  OSThread クラスのプラットフォーム依存な部分の定義
hotspot/src/os/linux/vm/osThread_linux.hpp               	  |  同上
hotspot/src/os/linux/vm/os_linux.cpp                     	  |  os クラスのプラットフォーム依存な部分の定義, 及びその補助クラスの定義 ([os::Linux, os::Linux::SuspendResume, os::PlatformEvent, os::PlatformParker](noQaAIxjzs.html))
hotspot/src/os/linux/vm/os_linux.hpp                              |  同上
hotspot/src/os/linux/vm/os_linux.inline.hpp              	  |  同上
hotspot/src/os/linux/vm/os_share_linux.hpp               	  |  os クラスのプラットフォーム依存な部分で使用する関数の宣言
hotspot/src/os/linux/vm/perfMemory_linux.cpp             	  |  PerfMemory クラスのプラットフォーム依存なメソッドの定義
hotspot/src/os/linux/vm/stubRoutines_linux.cpp                    |  空ファイル(※8)
hotspot/src/os/linux/vm/threadCritical_linux.cpp         	  |  ThreadCritical クラスのプラットフォーム依存なメソッドの定義
hotspot/src/os/linux/vm/thread_linux.inline.hpp          	  |  ThreadLocalStorage クラスのプラットフォーム依存なメソッドの定義(ThreadLocalStorage::pd_invalidate_all()) (※9)
hotspot/src/os/linux/vm/vmError_linux.cpp                	  |  VMError クラスのプラットフォーム依存なメソッドの定義 (VMError::show_message_box(), VMError::get_resetted_sighandler(), etc)

### 備考(Notes)
* ※1: 本来は C1 関連の JVM のコマンドラインオプションのうち OS 独自のものを定義するファイル(だと思われる). Linux ではそういったオプションが特にないため空ファイルとなっている模様.

* ※2: 本来は C2 関連の JVM のコマンドラインオプションのうち OS 独自のものを定義するファイル(だと思われる). Linux ではそういったオプションが特にないため空ファイルとなっている模様.

* ※3: どちらも何もしないメソッドとして定義されている.

```cpp
    ((cite: hotspot/src/os/linux/vm/chaitin_linux.cpp))
    void PhaseRegAlloc::pd_preallocate_hook() {
      // no action
    }
```

```cpp
    ((cite: hotspot/src/os/linux/vm/chaitin_linux.cpp))
    void PhaseRegAlloc::pd_postallocate_verify_hook() {
      // no action
    }
```

* ※4: 中身は abi::__cxa_demangle() を使用しているだけ.

```cpp
    ((cite: hotspot/src/os/linux/vm/decoder_linux.cpp))
    bool Decoder::demangle(const char* symbol, char *buf, int buflen) {
      int   status;
      char* result;
      size_t size = (size_t)buflen;
    
      // Don't pass buf to __cxa_demangle. In case of the 'buf' is too small,
      // __cxa_demangle will call system "realloc" for additional memory, which
      // may use different malloc/realloc mechanism that allocates 'buf'.
      if ((result = abi::__cxa_demangle(symbol, NULL, NULL, &status)) != NULL) {
        jio_snprintf(buf, buflen, "%s", result);
          // call c library's free
          ::free(result);
          return true;
      }
      return false;
    }
```

* ※5: (Linux 上ではサポートされないので) 何もしない関数として実装しているだけ.

* ※6: なお使う際には libc や libthread (libpthreadのtypo?) よりも先にロードする必要がある. また JVM_begin_signal_setting(), JVM_end_signal_setting(), JVM_get_signal_action() という関数も定義しているがこれらは使用箇所が見当たらない... #TODO.

```cpp
    ((cite: hotspot/src/os/linux/vm/jsig.c))
    /* This is a special library that should be loaded before libc &
     * libthread to interpose the signal handler installation functions:
     * sigaction(), signal(), sigset().
     * Used for signal-chaining. See RFE 4381843.
     */
```

* ※7: Mutex 関連のプラットフォーム依存な定義を入れるファイルだと思われるが #include 以外は何も書いていない.

* ※8: StubRoutines クラスのプラットフォーム依存なメソッドの定義を入れるファイルだと思われるが #include 以外は何も書いていない.

* ※9: 中身も何もしない関数として実装してあるだけ

```cpp
    ((cite: hotspot/src/os/linux/vm/thread_linux.inline.hpp))
    // Contains inlined functions for class Thread and ThreadLocalStorage
    
    inline void ThreadLocalStorage::pd_invalidate_all() {} // nothing to do
```







