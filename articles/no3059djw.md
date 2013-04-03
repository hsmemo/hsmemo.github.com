---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/os/windows/vm/os_windows.cpp
### 説明(description)

```
// Loads .dll/.so and
// in case of error it checks if .dll/.so was built for the
// same architecture as Hotspot is running on
```

### 名前(function name)
```
void * os::dll_load(const char *name, char *ebuf, int ebuflen)
{
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) LoadLibrary() が成功すればそれを返す.
      ---------------------------------------- -}

	  void * result = LoadLibrary(name);
	  if (result != NULL)
	  {
	    return result;
	  }
	
  {- -------------------------------------------
  (1) (以下は LoadLibrary() が失敗した場合の処理.
       失敗の原因を少し調べて分かりやすいエラーメッセージを返すようにしている.)
      ---------------------------------------- -}

	  long errcode = GetLastError();
	  if (errcode == ERROR_MOD_NOT_FOUND) {
	    strncpy(ebuf, "Can't find dependent libraries", ebuflen-1);
	    ebuf[ebuflen-1]='\0';
	    return NULL;
	  }
	
	  // Parsing dll below
	  // If we can read dll-info and find that dll was built
	  // for an architecture other than Hotspot is running in
	  // - then print to buffer "DLL was built for a different architecture"
	  // else call getLastErrorString to obtain system error message
	
	  // Read system error message into ebuf
	  // It may or may not be overwritten below (in the for loop and just above)
	  getLastErrorString(ebuf, (size_t) ebuflen);
	  ebuf[ebuflen-1]='\0';
	  int file_descriptor=::open(name, O_RDONLY | O_BINARY, 0);
	  if (file_descriptor<0)
	  {
	    return NULL;
	  }
	
	  uint32_t signature_offset;
	  uint16_t lib_arch=0;
	  bool failed_to_get_lib_arch=
	  (
	    //Go to position 3c in the dll
	    (os::seek_to_file_offset(file_descriptor,IMAGE_FILE_PTR_TO_SIGNATURE)<0)
	    ||
	    // Read loacation of signature
	    (sizeof(signature_offset)!=
	      (os::read(file_descriptor, (void*)&signature_offset,sizeof(signature_offset))))
	    ||
	    //Go to COFF File Header in dll
	    //that is located after"signature" (4 bytes long)
	    (os::seek_to_file_offset(file_descriptor,
	      signature_offset+IMAGE_FILE_SIGNATURE_LENGTH)<0)
	    ||
	    //Read field that contains code of architecture
	    // that dll was build for
	    (sizeof(lib_arch)!=
	      (os::read(file_descriptor, (void*)&lib_arch,sizeof(lib_arch))))
	  );
	
	  ::close(file_descriptor);
	  if (failed_to_get_lib_arch)
	  {
	    // file i/o error - report getLastErrorString(...) msg
	    return NULL;
	  }
	
	  typedef struct
	  {
	    uint16_t arch_code;
	    char* arch_name;
	  } arch_t;
	
	  static const arch_t arch_array[]={
	    {IMAGE_FILE_MACHINE_I386,      (char*)"IA 32"},
	    {IMAGE_FILE_MACHINE_AMD64,     (char*)"AMD 64"},
	    {IMAGE_FILE_MACHINE_IA64,      (char*)"IA 64"}
	  };
	  #if   (defined _M_IA64)
	    static const uint16_t running_arch=IMAGE_FILE_MACHINE_IA64;
	  #elif (defined _M_AMD64)
	    static const uint16_t running_arch=IMAGE_FILE_MACHINE_AMD64;
	  #elif (defined _M_IX86)
	    static const uint16_t running_arch=IMAGE_FILE_MACHINE_I386;
	  #else
	    #error Method os::dll_load requires that one of following \
	           is defined :_M_IA64,_M_AMD64 or _M_IX86
	  #endif
	
	
	  // Obtain a string for printf operation
	  // lib_arch_str shall contain string what platform this .dll was built for
	  // running_arch_str shall string contain what platform Hotspot was built for
	  char *running_arch_str=NULL,*lib_arch_str=NULL;
	  for (unsigned int i=0;i<ARRAY_SIZE(arch_array);i++)
	  {
	    if (lib_arch==arch_array[i].arch_code)
	      lib_arch_str=arch_array[i].arch_name;
	    if (running_arch==arch_array[i].arch_code)
	      running_arch_str=arch_array[i].arch_name;
	  }
	
	  assert(running_arch_str,
	    "Didn't find runing architecture code in arch_array");
	
	  // If the architure is right
	  // but some other error took place - report getLastErrorString(...) msg
	  if (lib_arch == running_arch)
	  {
	    return NULL;
	  }
	
	  if (lib_arch_str!=NULL)
	  {
	    ::_snprintf(ebuf, ebuflen-1,
	      "Can't load %s-bit .dll on a %s-bit platform",
	      lib_arch_str,running_arch_str);
	  }
	  else
	  {
	    // don't know what architecture this dll was build for
	    ::_snprintf(ebuf, ebuflen-1,
	      "Can't load this .dll (machine code=0x%x) on a %s-bit platform",
	      lib_arch,running_arch_str);
	  }
	
	  return NULL;
	}
	
```


