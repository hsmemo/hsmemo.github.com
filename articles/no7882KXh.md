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
klassOop instanceKlass::find_field(Symbol* name, Symbol* sig, fieldDescriptor* fd) const {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) JVMS に則った順序で探索を行う. 見つからなければ NULL をリターン.
      (1) 自分の中で探索
          instanceKlass::find_local_field()
      (1) スーパーインターフェースの中で探索
          instanceKlass::find_interface_field() 
      (1) スーパークラスの中で探索
          instanceKlass::find_field()  (super に対して再帰呼び出し)
      ---------------------------------------- -}

	  // search order according to newest JVM spec (5.4.3.2, p.167).
	  // 1) search for field in current klass
	  if (find_local_field(name, sig, fd)) {
	    return as_klassOop();
	  }
	  // 2) search for field recursively in direct superinterfaces
	  { klassOop intf = find_interface_field(name, sig, fd);
	    if (intf != NULL) return intf;
	  }
	  // 3) apply field lookup recursively if superclass exists
	  { klassOop supr = super();
	    if (supr != NULL) return instanceKlass::cast(supr)->find_field(name, sig, fd);
	  }
	  // 4) otherwise field lookup fails
	  return NULL;
	}
	
```


