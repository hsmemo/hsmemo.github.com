---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/arguments.cpp

### 名前(function name)
```
void Arguments::set_ergonomics_flags() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) DumpSharedSpaces オプションや RequireSharedSpaces オプションが指定されている場合は 
      Ergonomics によるオプション選択は行わない.
      ここでリターン.
      ---------------------------------------- -}

	  // Parallel GC is not compatible with sharing. If one specifies
	  // that they want sharing explicitly, do not set ergonomics flags.
	  if (DumpSharedSpaces || RequireSharedSpaces) {
	    return;
	  }
	
  {- -------------------------------------------
  (1) HotSpot が稼働している計算機の性能が高く(= os::is_server_class_machine() が true) で, 
      かつ, 明示的に client オプションが指定されておらず, 
      かつ, 明示的に GC アルゴリズムも指定されてなければ, 
      UseConcMarkSweepGC か UseParallelGC を true にする.
    
      * 停止時間を短く抑える必要がある場合 (= Arguments::should_auto_select_low_pause_collector() が true の場合): 
        UseConcMarkSweepGC を true にする.
      * そうではない場合: 
        UseParallelGC を true にする.
    
      (なお, この場合 UseSharedSpaces はオフにする.
       See: no_shared_spaces())
      ---------------------------------------- -}

	  if (os::is_server_class_machine() && !force_client_mode ) {
	    // If no other collector is requested explicitly,
	    // let the VM select the collector based on
	    // machine class and automatic selection policy.
	    if (!UseSerialGC &&
	        !UseConcMarkSweepGC &&
	        !UseG1GC &&
	        !UseParNewGC &&
	        !DumpSharedSpaces &&
	        FLAG_IS_DEFAULT(UseParallelGC)) {
	      if (should_auto_select_low_pause_collector()) {
	        FLAG_SET_ERGO(bool, UseConcMarkSweepGC, true);
	      } else {
	        FLAG_SET_ERGO(bool, UseParallelGC, true);
	      }
	      no_shared_spaces();
	    }
	  }
	
  {- -------------------------------------------
  (1) 以下で UseCompressedOops オプションの値を決定する.
      (UseCompressedOops オプションを使うとヒープサイズが 32GB 以下に制限されるので, 
      それを超えるヒープサイズが指定されている場合には使えない)
    
      (なお, この処理は zero の場合には行われない)
      (また, この処理は 64bit 環境 (#ifdef _LP64 時) の場合にしか行われない (<= というか 32bit ならやる意味が無い...))
      ---------------------------------------- -}

	#ifndef ZERO
	#ifdef _LP64
	  // Check that UseCompressedOops can be set with the max heap size allocated
	  // by ergonomics.

    {- -------------------------------------------
  (1.1) C2, Shark, または Tiered 使用時には, 
        MaxHeapSize が Compressed Oop 適用時の最大ヒープサイズ以下 (= max_heap_for_compressed_oops() 以下) に収まり, 
        さらに UseCompressedOops がデフォルト値のままなら,
        UseCompressedOops オプションを true にする.
    
        (また, 64bit Windows では, 
         MaxHeapSize が Compressed Oop 適用時の最大ヒープサイズ以下で, 
         さらに UseLargePages と UseCompressedOops の両方が true になれば
         Universe::_narrow_oop._use_implicit_null_checks を false にしておく.
         ただしこの値は, ヒープ領域が 0 アドレスを起点として確保できた場合には, 後で true に戻される.
         (See: Universe::initialize_heap()))
        ---------------------------------------- -}

	  if (MaxHeapSize <= max_heap_for_compressed_oops()) {
	#if !defined(COMPILER1) || defined(TIERED)
	    if (FLAG_IS_DEFAULT(UseCompressedOops)) {
	      FLAG_SET_ERGO(bool, UseCompressedOops, true);
	    }
	#endif
	#ifdef _WIN64
	    if (UseLargePages && UseCompressedOops) {
	      // Cannot allocate guard pages for implicit checks in indexed addressing
	      // mode, when large pages are specified on windows.
	      // This flag could be switched ON if narrow oop base address is set to 0,
	      // see code in Universe::initialize_heap().
	      Universe::set_narrow_oop_use_implicit_null_checks(false);
	    }
	#endif //  _WIN64

    {- -------------------------------------------
  (1.1) もし UseCompressedOops が明示的にセットされているが MaxHeapSize が大きすぎる場合には, 
        warning() を出した後, UseCompressedOops を false にする.
        ---------------------------------------- -}

	  } else {
	    if (UseCompressedOops && !FLAG_IS_DEFAULT(UseCompressedOops)) {
	      warning("Max heap size too large for Compressed Oops");
	      FLAG_SET_DEFAULT(UseCompressedOops, false);
	    }
	  }
	  // Also checks that certain machines are slower with compressed oops
	  // in vm_version initialization code.
	#endif // _LP64
	#endif // !ZERO
	}
	
```


