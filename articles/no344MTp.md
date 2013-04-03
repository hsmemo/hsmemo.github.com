---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/collectorPolicy.cpp
### 説明(description)

```
// Minimum sizes of the generations may be different than
// the initial sizes.  An inconsistently is permitted here
// in the total size that can be specified explicitly by
// command line specification of OldSize and NewSize and
// also a command line specification of -Xms.  Issue a warning
// but allow the values to pass.

```

### 名前(function name)
```
void TwoGenerationCollectorPolicy::initialize_size_info() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) GenCollectorPolicy::initialize_size_info() を呼んで
      フィールドの初期化を行う.
      ---------------------------------------- -}

	  GenCollectorPolicy::initialize_size_info();
	
  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // At this point the minimum, initial and maximum sizes
	  // of the overall heap and of gen0 have been determined.
	  // The maximum gen1 size can be determined from the maximum gen0
	  // and maximum heap size since no explicit flags exits
	  // for setting the gen1 maximum.
	  _max_gen1_size = max_heap_byte_size() - _max_gen0_size;
	  _max_gen1_size =
	    MAX2((uintx)align_size_down(_max_gen1_size, min_alignment()),
	         min_alignment());

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // If no explicit command line flag has been set for the
	  // gen1 size, use what is left for gen1.
	  if (FLAG_IS_DEFAULT(OldSize) || FLAG_IS_ERGO(OldSize)) {
	    // The user has not specified any value or ergonomics
	    // has chosen a value (which may or may not be consistent
	    // with the overall heap size).  In either case make
	    // the minimum, maximum and initial sizes consistent
	    // with the gen0 sizes and the overall heap sizes.
	    assert(min_heap_byte_size() > _min_gen0_size,
	      "gen0 has an unexpected minimum size");
	    set_min_gen1_size(min_heap_byte_size() - min_gen0_size());
	    set_min_gen1_size(
	      MAX2((uintx)align_size_down(_min_gen1_size, min_alignment()),
	           min_alignment()));
	    set_initial_gen1_size(initial_heap_byte_size() - initial_gen0_size());
	    set_initial_gen1_size(
	      MAX2((uintx)align_size_down(_initial_gen1_size, min_alignment()),
	           min_alignment()));
	
	  } else {
	    // It's been explicitly set on the command line.  Use the
	    // OldSize and then determine the consequences.
	    set_min_gen1_size(OldSize);
	    set_initial_gen1_size(OldSize);
	
    {- -------------------------------------------
  (1.1) (warning)
        ---------------------------------------- -}

	    // If the user has explicitly set an OldSize that is inconsistent
	    // with other command line flags, issue a warning.
	    // The generation minimums and the overall heap mimimum should
	    // be within one heap alignment.
	    if ((_min_gen1_size + _min_gen0_size + min_alignment()) <
	           min_heap_byte_size()) {
	      warning("Inconsistency between minimum heap size and minimum "
	          "generation sizes: using minimum heap = " SIZE_FORMAT,
	          min_heap_byte_size());
	    }
	    if ((OldSize > _max_gen1_size)) {
	      warning("Inconsistency between maximum heap size and maximum "
	          "generation sizes: using maximum heap = " SIZE_FORMAT
	          " -XX:OldSize flag is being ignored",
	          max_heap_byte_size());
	    }

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // If there is an inconsistency between the OldSize and the minimum and/or
	    // initial size of gen0, since OldSize was explicitly set, OldSize wins.
	    if (adjust_gen0_sizes(&_min_gen0_size, &_min_gen1_size,
	                          min_heap_byte_size(), OldSize)) {
	      if (PrintGCDetails && Verbose) {
	        gclog_or_tty->print_cr("2: Minimum gen0 " SIZE_FORMAT "  Initial gen0 "
	              SIZE_FORMAT "  Maximum gen0 " SIZE_FORMAT,
	              min_gen0_size(), initial_gen0_size(), max_gen0_size());
	      }
	    }

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    // Initial size
	    if (adjust_gen0_sizes(&_initial_gen0_size, &_initial_gen1_size,
	                         initial_heap_byte_size(), OldSize)) {
	      if (PrintGCDetails && Verbose) {
	        gclog_or_tty->print_cr("3: Minimum gen0 " SIZE_FORMAT "  Initial gen0 "
	          SIZE_FORMAT "  Maximum gen0 " SIZE_FORMAT,
	          min_gen0_size(), initial_gen0_size(), max_gen0_size());
	      }
	    }
	  }

  {- -------------------------------------------
  (1) フィールドの初期化
      ---------------------------------------- -}

	  // Enforce the maximum gen1 size.
	  set_min_gen1_size(MIN2(_min_gen1_size, _max_gen1_size));
	
	  // Check that min gen1 <= initial gen1 <= max gen1
	  set_initial_gen1_size(MAX2(_initial_gen1_size, _min_gen1_size));
	  set_initial_gen1_size(MIN2(_initial_gen1_size, _max_gen1_size));
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintGCDetails && Verbose) {
	    gclog_or_tty->print_cr("Minimum gen1 " SIZE_FORMAT "  Initial gen1 "
	      SIZE_FORMAT "  Maximum gen1 " SIZE_FORMAT,
	      min_gen1_size(), initial_gen1_size(), max_gen1_size());
	  }
	}
	
```


