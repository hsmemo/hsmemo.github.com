---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/compilationPolicy.cpp

### 名前(function name)
```
void CompilationPolicy::completed_vm_startup() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (TraceCompilationPolicy) {
	    tty->print("CompilationPolicy: completed vm startup.\n");
	  }

  {- -------------------------------------------
  (1) _in_vm_startup フィールドの値を false にする.
    
      (この値は CompilationPolicy::delay_compilation_during_startup() で参照される.
       See: CompilationPolicy::delay_compilation_during_startup())
      ---------------------------------------- -}

	  _in_vm_startup = false;
	}
	
```


