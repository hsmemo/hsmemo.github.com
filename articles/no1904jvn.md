---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/reflectionUtils.hpp

### 名前(function name)
```
  void next() {
```

### 本体部(body)
```
	    _index -= instanceKlass::next_offset;
	    if (has_filtered_field()) {
	      while (_index >=0 && FilteredFieldsMap::is_filtered_field((klassOop)_klass(), offset())) {
	        _index -= instanceKlass::next_offset;
	      }
	    }
	  }
	
```


