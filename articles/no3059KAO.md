---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp
### 説明(description)

```
// set_lwp_priority
//
// Set the priority of the lwp.  This call should only be made
// when using bound threads (T2 threads are bound by default).
//
```

### 名前(function name)
```
int     set_lwp_priority (int ThreadID, int lwpid, int newPrio )
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int rslt;
	  int Actual, Expected, prv;
	  pcparms_t ParmInfo;                   // for GET-SET
	#ifdef ASSERT
	  pcparms_t ReadBack;                   // for readback
	#endif
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // Set priority via PC_GETPARMS, update, PC_SETPARMS
	  // Query current values.
	  // TODO: accelerate this by eliminating the PC_GETPARMS call.
	  // Cache "pcparms_t" in global ParmCache.
	  // TODO: elide set-to-same-value
	
  {- -------------------------------------------
  (1) もし初期化時に失敗していたら (= priocntl_enable 変数の値が false であれば), ここでリターン.
      (See: lwp_priocntl_init())
    
      (ついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  // If something went wrong on init, don't change priorities.
	  if ( !priocntl_enable ) {
	    if (ThreadPriorityVerbose)
	      tty->print_cr("Trying to set priority but init failed, ignoring");
	    return EINVAL;
	  }
	
	
  {- -------------------------------------------
  (1) 引数で指定された lwp id が不正な場合, ここでリターン.
      (ついでに(トレース出力)も出している)
      ---------------------------------------- -}

	  // If lwp hasn't started yet, just return
	  // the _start routine will call us again.
	  if ( lwpid <= 0 ) {
	    if (ThreadPriorityVerbose) {
	      tty->print_cr ("deferring the set_lwp_priority of thread " INTPTR_FORMAT " to %d, lwpid not set",
	                     ThreadID, newPrio);
	    }
	    return 0;
	  }
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (ThreadPriorityVerbose) {
	    tty->print_cr ("set_lwp_priority(" INTPTR_FORMAT "@" INTPTR_FORMAT " %d) ",
	                   ThreadID, lwpid, newPrio);
	  }
	
  {- -------------------------------------------
  (1) (priocntl_ptr 変数経由で) priocntl() を呼び出して, 優先度情報(pcparms_t型のデータ)を取得する.
      失敗したらここでリターン.
      ---------------------------------------- -}

	  memset(&ParmInfo, 0, sizeof(pcparms_t));
	  ParmInfo.pc_cid = PC_CLNULL;
	  rslt = (*priocntl_ptr)(PC_VERSION, P_LWPID, lwpid, PC_GETPARMS, (caddr_t)&ParmInfo);
	  if (rslt < 0) return errno;
	
  {- -------------------------------------------
  (1) 取得した優先度情報を変更して, 新しい優先度に対応する優先度情報(pcparms_t型のデータ)を作成.
      ---------------------------------------- -}

	  if (ParmInfo.pc_cid == rtLimits.schedPolicy) {
	    rtparms_t *rtInfo  = (rtparms_t*)ParmInfo.pc_clparms;
	    rtInfo->rt_pri     = scale_to_lwp_priority (rtLimits.minPrio, rtLimits.maxPrio, newPrio);
	    rtInfo->rt_tqsecs  = RT_NOCHANGE;
	    rtInfo->rt_tqnsecs = RT_NOCHANGE;
	    if (ThreadPriorityVerbose) {
	      tty->print_cr("RT: %d->%d\n", newPrio, rtInfo->rt_pri);
	    }
	  } else if (ParmInfo.pc_cid == iaLimits.schedPolicy) {
	    iaparms_t *iaInfo  = (iaparms_t*)ParmInfo.pc_clparms;
	    int maxClamped     = MIN2(iaLimits.maxPrio, (int)iaInfo->ia_uprilim);
	    iaInfo->ia_upri    = scale_to_lwp_priority(iaLimits.minPrio, maxClamped, newPrio);
	    iaInfo->ia_uprilim = IA_NOCHANGE;
	    iaInfo->ia_mode    = IA_NOCHANGE;
	    if (ThreadPriorityVerbose) {
	      tty->print_cr ("IA: [%d...%d] %d->%d\n",
	               iaLimits.minPrio, maxClamped, newPrio, iaInfo->ia_upri);
	    }
	  } else if (ParmInfo.pc_cid == tsLimits.schedPolicy) {
	    tsparms_t *tsInfo  = (tsparms_t*)ParmInfo.pc_clparms;
	    int maxClamped     = MIN2(tsLimits.maxPrio, (int)tsInfo->ts_uprilim);
	    prv                = tsInfo->ts_upri;
	    tsInfo->ts_upri    = scale_to_lwp_priority(tsLimits.minPrio, maxClamped, newPrio);
	    tsInfo->ts_uprilim = IA_NOCHANGE;
	    if (ThreadPriorityVerbose) {
	      tty->print_cr ("TS: %d [%d...%d] %d->%d\n",
	               prv, tsLimits.minPrio, maxClamped, newPrio, tsInfo->ts_upri);
	    }
	    if (prv == tsInfo->ts_upri) return 0;
	  } else {
	    if ( ThreadPriorityVerbose ) {
	      tty->print_cr ("Unknown scheduling class\n");
	    }
	      return EINVAL;    // no clue, punt
	  }
	
  {- -------------------------------------------
  (1) (priocntl_ptr 変数経由で) priocntl() を呼び出して, 優先度を変更する.
      失敗したらここでリターン.
      ---------------------------------------- -}

	  rslt = (*priocntl_ptr)(PC_VERSION, P_LWPID, lwpid, PC_SETPARMS, (caddr_t)&ParmInfo);
	  if (ThreadPriorityVerbose && rslt) {
	    tty->print_cr ("PC_SETPARMS ->%d %d\n", rslt, errno);
	  }
	  if (rslt < 0) return errno;
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  // Sanity check: read back what we just attempted to set.
	  // In theory it could have changed in the interim ...
	  //
	  // The priocntl system call is tricky.
	  // Sometimes it'll validate the priority value argument and
	  // return EINVAL if unhappy.  At other times it fails silently.
	  // Readbacks are prudent.
	
	  if (!ReadBackValidate) return 0;
	
	  memset(&ReadBack, 0, sizeof(pcparms_t));
	  ReadBack.pc_cid = PC_CLNULL;
	  rslt = (*priocntl_ptr)(PC_VERSION, P_LWPID, lwpid, PC_GETPARMS, (caddr_t)&ReadBack);
	  assert(rslt >= 0, "priocntl failed");
	  Actual = Expected = 0xBAD;
	  assert(ParmInfo.pc_cid == ReadBack.pc_cid, "cid's don't match");
	  if (ParmInfo.pc_cid == rtLimits.schedPolicy) {
	    Actual   = RTPRI(ReadBack)->rt_pri;
	    Expected = RTPRI(ParmInfo)->rt_pri;
	  } else if (ParmInfo.pc_cid == iaLimits.schedPolicy) {
	    Actual   = IAPRI(ReadBack)->ia_upri;
	    Expected = IAPRI(ParmInfo)->ia_upri;
	  } else if (ParmInfo.pc_cid == tsLimits.schedPolicy) {
	    Actual   = TSPRI(ReadBack)->ts_upri;
	    Expected = TSPRI(ParmInfo)->ts_upri;
	  } else {
	    if ( ThreadPriorityVerbose ) {
	      tty->print_cr("set_lwp_priority: unexpected class in readback: %d\n", ParmInfo.pc_cid);
	    }
	  }
	
	  if (Actual != Expected) {
	    if ( ThreadPriorityVerbose ) {
	      tty->print_cr ("set_lwp_priority(%d %d) Class=%d: actual=%d vs expected=%d\n",
	             lwpid, newPrio, ReadBack.pc_cid, Actual, Expected);
	    }
	  }
	#endif
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return 0;
	}
	
```


