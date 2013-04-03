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
int os::Linux::safe_cond_timedwait(pthread_cond_t *_cond, pthread_mutex_t *_mutex, const struct timespec *_abstime)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) pthread_cond_timedwait() で眠りにつくだけ.
  
      (ただし, LinuxThreads の場合には FPU の control word をリセットしてくれるので, 
       pthread_cond_timedwait() 呼び出しの前後で待避／復帰処理を行っている.)
      ---------------------------------------- -}

	   if (is_NPTL()) {
	      return pthread_cond_timedwait(_cond, _mutex, _abstime);
	   } else {
	#ifndef IA64
	      // 6292965: LinuxThreads pthread_cond_timedwait() resets FPU control
	      // word back to default 64bit precision if condvar is signaled. Java
	      // wants 53bit precision.  Save and restore current value.
	      int fpu = get_fpu_control_word();
	#endif // IA64
	      int status = pthread_cond_timedwait(_cond, _mutex, _abstime);
	#ifndef IA64
	      set_fpu_control_word(fpu);
	#endif // IA64
	      return status;
	   }
	}
	
```


