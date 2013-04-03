---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/os_cpu/ 以下 ： linux_x86/
---
[Up](noNPfVR_fz.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/os_cpu/ 以下 ： linux_x86/

--- 

File Name                                                         | Description
----------------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/os_cpu/linux_x86/vm/assembler_linux_x86.cpp           |  MacroAssembler クラスのプラットフォーム依存なメソッドの定義 (MacroAssembler::int3(), MacroAssembler::get_thread())
hotspot/src/os_cpu/linux_x86/vm/atomic_linux_x86.inline.hpp       |  Atomic クラスのプラットフォーム依存なメソッドの定義
hotspot/src/os_cpu/linux_x86/vm/bytes_linux_x86.inline.hpp        |  Bytes クラスのプラットフォーム依存なメソッドの定義 (Bytes::swap_u2(), Bytes::swap_u4(), Bytes::swap_u8())
hotspot/src/os_cpu/linux_x86/vm/copy_linux_x86.inline.hpp         |  Copy クラスのプラットフォーム依存なメソッドの定義 (pd_conjoint_words(), pd_disjoint_words(), etc)
hotspot/src/os_cpu/linux_x86/vm/globals_linux_x86.hpp             |  Linux/x86 環境独自の JVM のコマンドラインオプションの定義 (あるいはデフォルト値の変更) (See: [here](no7882y_Y.html) for details)
hotspot/src/os_cpu/linux_x86/vm/linux_x86_32.ad                   |  AD ファイル (See: [here](noVxQtU9lk.html) for details)
hotspot/src/os_cpu/linux_x86/vm/linux_x86_32.s                    |  アセンブリで生成した関数の定義(x86/32bit 用) (Copy_conjoint_bytes, Copy_arrayof_conjoint_bytes, Copy_conjoint_jshorts_atomic, Copy_arrayof_conjoint_jshorts, Copy_conjoint_jints_atomic, Copy_arrayof_conjoint_jints, Copy_conjoint_jlongs_atomic, mmx_Copy_arrayof_conjoint_jshorts, Atomic_cmpxchg_long, Atomic_move_long, SafeFetch32, Fetch32PFI, Fetch32Resume, SafeFetchN, SpinPause) (※1)
hotspot/src/os_cpu/linux_x86/vm/linux_x86_64.ad                   |  AD ファイル (See: [here](noVxQtU9lk.html) for details)
hotspot/src/os_cpu/linux_x86/vm/linux_x86_64.s                    |  アセンブリで生成した関数の定義(x86/64bit 用) (_Copy_arrayof_conjoint_bytes, _Copy_arrayof_conjoint_jshorts, _Copy_conjoint_jshorts_atomic, _Copy_arrayof_conjoint_jints, _Copy_conjoint_jints_atomic, _Copy_arrayof_conjoint_jlongs, _Copy_conjoint_jlongs_atomic, SafeFetch32, Fetch32PFI, Fetch32Resume, SafeFetchN, FetchNPFI, FetchNResume, SpinPause) (※1)
hotspot/src/os_cpu/linux_x86/vm/orderAccess_linux_x86.inline.hpp  |  OrderAccess クラスのプラットフォーム依存なメソッドの定義 (OrderAccess::loadload(), OrderAccess::storestore(), OrderAccess::loadstore(), OrderAccess::storeload(), OrderAccess::acquire(), 等々多数)
hotspot/src/os_cpu/linux_x86/vm/os_linux_x86.cpp                  |  os クラスのプラットフォーム依存な部分の定義, 及び os::Linux クラスのメソッドの定義
hotspot/src/os_cpu/linux_x86/vm/os_linux_x86.hpp                  |  同上
hotspot/src/os_cpu/linux_x86/vm/prefetch_linux_x86.inline.hpp     |  Prefetch クラスのプラットフォーム依存なメソッドの定義 (Prefetch::read(), Prefetch::write())
hotspot/src/os_cpu/linux_x86/vm/threadLS_linux_x86.cpp            |  ThreadLocalStorage クラスのプラットフォーム依存な部分の定義
hotspot/src/os_cpu/linux_x86/vm/threadLS_linux_x86.hpp            |  同上
hotspot/src/os_cpu/linux_x86/vm/thread_linux_x86.cpp              |  JavaThread クラスのプラットフォーム依存な部分の定義
hotspot/src/os_cpu/linux_x86/vm/thread_linux_x86.hpp              |  同上
hotspot/src/os_cpu/linux_x86/vm/vmStructs_linux_x86.hpp           |  VMStructs クラス用のプラットフォーム依存なマクロの定義 (VM_STRUCTS_OS_CPU, VM_TYPES_OS_CPU, VM_INT_CONSTANTS_OS_CPU, VM_LONG_CONSTANTS_OS_CPU)
hotspot/src/os_cpu/linux_x86/vm/vm_version_linux_x86.cpp          |  (空ファイル)

### 備考(Notes)
* ※1: Copy クラスや Atomic クラスのプラットフォーム依存なメソッドのうち, アセンブリで書いた方がよいもの等がここに納められている.






