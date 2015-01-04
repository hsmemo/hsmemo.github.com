---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp
### 説明(description)

```
// Note: implementation moved to static method to expose the this pointer.
```

### 名前(function name)
```
void instanceKlass::set_initialization_state_and_notify(ClassState state, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) instanceKlass::set_initialization_state_and_notify_impl() を呼びだすだけ.
      ---------------------------------------- -}

	  instanceKlassHandle kh(THREAD, this->as_klassOop());
	  set_initialization_state_and_notify_impl(kh, state, CHECK);
	}
	
```


