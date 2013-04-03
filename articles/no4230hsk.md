---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/code/relocInfo.cpp

### 名前(function name)
```
void CallRelocation::fix_relocation_after_move(const CodeBuffer* src, CodeBuffer* dest) {
```

### 本体部(body)
```
	  // Usually a self-relative reference to an external routine.
	  // On some platforms, the reference is absolute (not self-relative).
	  // The enhanced use of pd_call_destination sorts this all out.
	  address orig_addr = old_addr_for(addr(), src, dest);
	  address callee    = pd_call_destination(orig_addr);
	  // Reassert the callee address, this time in the new copy of the code.
	  pd_set_call_destination(callee);
	
```


