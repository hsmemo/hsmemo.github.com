---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/linux/vm/os_linux.cpp

### 名前(function name)
```
jlong os::javaTimeNanos() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) clock_gettime() が使用できるなら, Linux::clock_gettime() で取得した値を返す.
      そうでなければ, gettimeofday() で取得した値を返す.
      ---------------------------------------- -}

	  if (Linux::supports_monotonic_clock()) {
	    struct timespec tp;
	    int status = Linux::clock_gettime(CLOCK_MONOTONIC, &tp);
	    assert(status == 0, "gettime error");
	    jlong result = jlong(tp.tv_sec) * (1000 * 1000 * 1000) + jlong(tp.tv_nsec);
	    return result;
	  } else {
	    timeval time;
	    int status = gettimeofday(&time, NULL);
	    assert(status != -1, "linux error");
	    jlong usecs = jlong(time.tv_sec) * (1000 * 1000) + jlong(time.tv_usec);
	    return 1000 * usecs;
	  }
	}
	
```


