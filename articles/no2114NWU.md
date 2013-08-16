---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/solaris/vm/os_solaris.cpp

### 名前(function name)
```
static int os_sleep(jlong millis, bool interruptible) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  const jlong limit = INT_MAX;
	  jlong prevtime;
	  int res;
	
  {- -------------------------------------------
  (1) 指定されたスリープ時間が INT_MAX を超えている場合には, 
      INT_MAX 分ずつに分けてスリープ処理を行う.
      (この関数自身を再帰呼び出しし, INT_MAX 時間だけのスリープ処理を行う.
       この処理を, 残りのスリープ時間が INT_MAX 以下になるまで繰り返す.
  
       ただし, 途中で java.lang.Thread.interrupt() によって
       割り込まれた場合(= 返値が OS_OK ではなかった場合) には
       その時点で処理を終えてリターン)
      ---------------------------------------- -}

	  while (millis > limit) {
	    if ((res = os_sleep(limit, interruptible)) != OS_OK)
	      return res;
	    millis -= limit;
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Restart interrupted polls with new parameters until the proper delay
	  // has been completed.
	
	  prevtime = getTimeMillis();
	
  {- -------------------------------------------
  (1) (以下の while ループ内で, 指定されたスリープ時間が経過するまで寝続ける.
    
       眠る方法は, java.lang.Thread.interrupt() による割り込みを
       許すかどうか(= 引数の interruptible が true か否か)に応じて2通り.
       * java.lang.Thread.interrupt() による割り込みを許す場合
         単に poll() で寝るだけ.
       * java.lang.Thread.interrupt() による割り込みを許さない場合
         同じく poll() で寝るが, INTERRUPTIBLE_NORESTART_VM_ALWAYS マクロを使って
         poll() が割り込まれたら返り値(以下の res)に OS_INTRPT という値が入るようなコードにしている.
  
       寝ている間に java.lang.Thread.interrupt() で割り込まれたのかもしれないので, 
       目が覚める度に res の値をチェックしている.
       もし割り込まれていれば, その時点でリターンする (OS_INTRPT をリターン).
       (See: java.lang.Thread.interrupt())
  
       なお, 予定より早く起きることもあり得るので, 
       目が覚める度に getTimeMillis() で時間を確認し, 
       ちゃんと指定時間分だけ経過するまで poll() の呼び出しを繰り返している.)
      ---------------------------------------- -}

	  while (millis > 0) {
	    jlong newtime;
	
	    if (!interruptible) {
	      // Following assert fails for os::yield_all:
	      // assert(!thread->is_Java_thread(), "must not be java thread");
	      res = poll(NULL, 0, millis);
	    } else {
	      JavaThread *jt = JavaThread::current();
	
	      INTERRUPTIBLE_NORESTART_VM_ALWAYS(poll(NULL, 0, millis), res, jt,
	        os::Solaris::clear_interrupted);
	    }
	
	    // INTERRUPTIBLE_NORESTART_VM_ALWAYS returns res == OS_INTRPT for
	    // thread.Interrupt.
	
	    // See c/r 6751923. Poll can return 0 before time
	    // has elapsed if time is set via clock_settime (as NTP does).
	    // res == 0 if poll timed out (see man poll RETURN VALUES)
	    // using the logic below checks that we really did
	    // sleep at least "millis" if not we'll sleep again.
	    if( ( res == 0 ) || ((res == OS_ERR) && (errno == EINTR))) {
	      newtime = getTimeMillis();
	      assert(newtime >= prevtime, "time moving backwards");
	    /* Doing prevtime and newtime in microseconds doesn't help precision,
	       and trying to round up to avoid lost milliseconds can result in a
	       too-short delay. */
	      millis -= newtime - prevtime;
	      if(millis <= 0)
	        return OS_OK;
	      prevtime = newtime;
	    } else
	      return res;
	  }
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return OS_OK;
	}
	
```


