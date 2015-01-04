---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/instanceKlass.cpp

### 名前(function name)
```
klassOop instanceKlass::find_interface_field(Symbol* name, Symbol* sig, fieldDescriptor* fd) const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) local_interfaces() の中を順に辿り, instanceKlass::find_local_field() と instanceKlass::find_interface_field() で調べていく.
      (まず instanceKlass::find_local_field() で調べ, ダメなら instanceKlass::find_interface_field() で再帰的に調べる.
       それでもダメなら次のインターフェースにうつる.)
      見つからなければ NULL をリターン.
      ---------------------------------------- -}

	  const int n = local_interfaces()->length();
	  for (int i = 0; i < n; i++) {
	    klassOop intf1 = klassOop(local_interfaces()->obj_at(i));
	    assert(Klass::cast(intf1)->is_interface(), "just checking type");
	    // search for field in current interface
	    if (instanceKlass::cast(intf1)->find_local_field(name, sig, fd)) {
	      assert(fd->is_static(), "interface field must be static");
	      return intf1;
	    }
	    // search for field in direct superinterfaces
	    klassOop intf2 = instanceKlass::cast(intf1)->find_interface_field(name, sig, fd);
	    if (intf2 != NULL) return intf2;
	  }
	  // otherwise field lookup fails
	  return NULL;
	}
	
```


