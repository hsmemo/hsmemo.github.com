---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/utilities/vmError.cpp

### 名前(function name)
```
void VM_ReportJavaOutOfMemory::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // Don't allocate large buffer on stack
	  static char buffer[O_BUFLEN];
	
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  tty->print_cr("#");
	  tty->print_cr("# java.lang.OutOfMemoryError: %s", _err->message());
	  tty->print_cr("# -XX:OnOutOfMemoryError=\"%s\"", OnOutOfMemoryError);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // make heap parsability
	  Universe::heap()->ensure_parsability(false);  // no need to retire TLABs
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  char* cmd;
	  const char* ptr = OnOutOfMemoryError;

  {- -------------------------------------------
  (1) (以下の while ループ内で, OnError オプションで指定されたコマンドを全て実行する)
      ---------------------------------------- -}

	  while ((cmd = next_OnError_command(buffer, sizeof(buffer), &ptr)) != NULL){

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    tty->print("#   Executing ");
	#if defined(LINUX)
	    tty->print  ("/bin/sh -c ");
	#elif defined(SOLARIS)
	    tty->print  ("/usr/bin/sh -c ");
	#endif
	    tty->print_cr("\"%s\"...", cmd);
	
    {- -------------------------------------------
  (1.1) os::fork_and_exec() を呼んで指定されたコマンドを実行する.
        ---------------------------------------- -}

	    os::fork_and_exec(cmd);
	
```


