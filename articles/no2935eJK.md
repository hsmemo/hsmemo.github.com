---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/assembler_x86.hpp

### 名前(function name)
```
  static relocInfo::relocType reloc_for_target(address target) {
```

### 本体部(body)
```
	    // Sometimes ExternalAddress is used for values which aren't
	    // exactly addresses, like the card table base.
	    // external_word_type can't be used for values in the first page
	    // so just skip the reloc in that case.
	    return external_word_Relocation::can_be_relocated(target) ? relocInfo::external_word_type : relocInfo::none;
	  }
	
```


