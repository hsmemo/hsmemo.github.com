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
jlong os::thread_cpu_time(Thread *thread, bool user_sys_cpu_time) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) proc ファイルシステム (の /proc/<pid>/lwp/<tid>/lwpusage) から CPU 使用時間の情報を取得する.
      ---------------------------------------- -}

	  char proc_name[64];
	  int count;
	  prusage_t prusage;
	  jlong lwp_time;
	  int fd;
	
	  sprintf(proc_name, "/proc/%d/lwp/%d/lwpusage",
	                     getpid(),
	                     thread->osthread()->lwp_id());
	  fd = ::open(proc_name, O_RDONLY);
	  if ( fd == -1 ) return -1;
	
	  do {
	    count = ::pread(fd,
	                  (void *)&prusage.pr_utime,
	                  thr_time_size,
	                  thr_time_off);
	  } while (count < 0 && errno == EINTR);
	  ::close(fd);
	  if ( count < 0 ) return -1;
	
	  if (user_sys_cpu_time) {
	    // user + system CPU time
	    lwp_time = (((jlong)prusage.pr_stime.tv_sec +
	                 (jlong)prusage.pr_utime.tv_sec) * (jlong)1000000000) +
	                 (jlong)prusage.pr_stime.tv_nsec +
	                 (jlong)prusage.pr_utime.tv_nsec;
	  } else {
	    // user level CPU time only
	    lwp_time = ((jlong)prusage.pr_utime.tv_sec * (jlong)1000000000) +
	                (jlong)prusage.pr_utime.tv_nsec;
	  }
	
	  return(lwp_time);
	}
	
```


