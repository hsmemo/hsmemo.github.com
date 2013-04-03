---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/vmreg_sparc.cpp

### 名前(function name)
```
void VMRegImpl::set_regName() {
```

### 本体部(body)
```
	  Register reg = ::as_Register(0);
	  int i;
	  for (i = 0; i < ConcreteRegisterImpl::max_gpr ; ) {
	    regName[i++  ] = reg->name();
	    regName[i++  ] = reg->name();
	    reg = reg->successor();
	  }
	
	  FloatRegister freg = ::as_FloatRegister(0);
	  for ( ; i < ConcreteRegisterImpl::max_fpr ; ) {
	    regName[i++] = freg->name();
	    if (freg->encoding() > 31) {
	      regName[i++] = freg->name();
	    }
	    freg = freg->successor();
	  }
	
	  for ( ; i < ConcreteRegisterImpl::number_of_registers ; i ++ ) {
	    regName[i] = "NON-GPR-FPR";
	  }
	}
	
```


