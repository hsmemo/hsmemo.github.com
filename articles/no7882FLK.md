---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/bytecodeInterpreter.cpp
### 説明(description)

```
/*
 * For those opcodes that need to have a GC point on a backwards branch
 */

// Backedge counting is kind of strange. The asm interpreter will increment
// the backedge counter as a separate counter but it does it's comparisons
// to the sum (scaled) of invocation counter and backedge count to make
// a decision. Seems kind of odd to sum them together like that

// skip is delta from current bcp/bci for target, branch_pc is pre-branch bcp


```

### 名前(function name)
```
#define DO_BACKEDGE_CHECKS(skip, branch_pc)                                                         \
```

### 本体部(body)
```
	    if ((skip) <= 0) {                                                                              \
	      if (UseLoopCounter) {                                                                         \
	        bool do_OSR = UseOnStackReplacement;                                                        \
	        BACKEDGE_COUNT->increment();                                                                \
	        if (do_OSR) do_OSR = BACKEDGE_COUNT->reached_InvocationLimit();                             \
	        if (do_OSR) {                                                                               \
	          nmethod*  osr_nmethod;                                                                    \
	          OSR_REQUEST(osr_nmethod, branch_pc);                                                      \
	          if (osr_nmethod != NULL && osr_nmethod->osr_entry_bci() != InvalidOSREntryBci) {          \
	            intptr_t* buf = SharedRuntime::OSR_migration_begin(THREAD);                             \
	            istate->set_msg(do_osr);                                                                \
	            istate->set_osr_buf((address)buf);                                                      \
	            istate->set_osr_entry(osr_nmethod->osr_entry());                                        \
	            return;                                                                                 \
	          }                                                                                         \
	        }                                                                                           \
	      }  /* UseCompiler ... */                                                                      \
	      INCR_INVOCATION_COUNT;                                                                        \
	      SAFEPOINT;                                                                                    \
	    }
	
```


