---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/solaris/bin/java_md.c

### 名前(function name)
```
jboolean
LoadJavaVM(const char *jvmpath, InvocationFunctions *ifn)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	    Dl_info dlinfo;
	    void *libjvm;
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	    JLI_TraceLauncher("JVM path is %s\n", jvmpath);
	
  {- -------------------------------------------
  (1) libjvm を dlopen で開く.
      ---------------------------------------- -}

	    libjvm = dlopen(jvmpath, RTLD_NOW + RTLD_GLOBAL);

  {- -------------------------------------------
  (1) もし開けなかったらエラーを出力して, JNI_FALSE を return
      ---------------------------------------- -}

	    if (libjvm == NULL) {
	#if defined(__solaris__) && defined(__sparc) && !defined(_LP64) /* i.e. 32-bit sparc */
	      FILE * fp;
	      Elf32_Ehdr elf_head;
	      int count;
	      int location;
	
	      fp = fopen(jvmpath, "r");
	      if (fp == NULL) {
	        JLI_ReportErrorMessage(DLL_ERROR2, jvmpath, dlerror());
	        return JNI_FALSE;
	      }
	
	      /* read in elf header */
	      count = fread((void*)(&elf_head), sizeof(Elf32_Ehdr), 1, fp);
	      fclose(fp);
	      if (count < 1) {
	        JLI_ReportErrorMessage(DLL_ERROR2, jvmpath, dlerror());
	        return JNI_FALSE;
	      }
	
	      /*
	       * Check for running a server vm (compiled with -xarch=v8plus)
	       * on a stock v8 processor.  In this case, the machine type in
	       * the elf header would not be included the architecture list
	       * provided by the isalist command, which is turn is gotten from
	       * sysinfo.  This case cannot occur on 64-bit hardware and thus
	       * does not have to be checked for in binaries with an LP64 data
	       * model.
	       */
	      if (elf_head.e_machine == EM_SPARC32PLUS) {
	        char buf[257];  /* recommended buffer size from sysinfo man
	                           page */
	        long length;
	        char* location;
	
	        length = sysinfo(SI_ISALIST, buf, 257);
	        if (length > 0) {
	            location = JLI_StrStr(buf, "sparcv8plus ");
	          if (location == NULL) {
	            JLI_ReportErrorMessage(JVM_ERROR3);
	            return JNI_FALSE;
	          }
	        }
	      }
	#endif
	        JLI_ReportErrorMessage(DLL_ERROR1, __LINE__);
	        JLI_ReportErrorMessage(DLL_ERROR2, jvmpath, dlerror());
	        return JNI_FALSE;
	    }
	
  {- -------------------------------------------
  (1) JNI_CreateJavaVM() を dlsym で取得する
      ---------------------------------------- -}

	    ifn->CreateJavaVM = (CreateJavaVM_t)
	        dlsym(libjvm, "JNI_CreateJavaVM");
	    if (ifn->CreateJavaVM == NULL) {
	        JLI_ReportErrorMessage(DLL_ERROR2, jvmpath, dlerror());
	        return JNI_FALSE;
	    }
	
  {- -------------------------------------------
  (1) JNI_GetDefaultJavaVMInitArgs() を dlsym で取得する.
      ---------------------------------------- -}

	    ifn->GetDefaultJavaVMInitArgs = (GetDefaultJavaVMInitArgs_t)
	        dlsym(libjvm, "JNI_GetDefaultJavaVMInitArgs");
	    if (ifn->GetDefaultJavaVMInitArgs == NULL) {
	        JLI_ReportErrorMessage(DLL_ERROR2, jvmpath, dlerror());
	        return JNI_FALSE;
	    }
	
	    return JNI_TRUE;
	}
	
```


