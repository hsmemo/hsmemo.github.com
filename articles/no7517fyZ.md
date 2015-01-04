---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/systemDictionary.cpp
### 説明(description)

```
// This routine does not lock the system dictionary.
//
// Since readers don't hold a lock, we must make sure that system
// dictionary entries are only removed at a safepoint (when only one
// thread is running), and are added to in a safe way (all links must
// be updated in an MT-safe manner).
//
// Callers should be aware that an entry could be added just after
// _dictionary->bucket(index) is read here, so the caller will not see
// the new entry.

```

### 名前(function name)
```
klassOop SystemDictionary::find(Symbol* class_name,
                                Handle class_loader,
                                Handle protection_domain,
                                TRAPS) {
```

### 本体部(body)
```
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // UseNewReflection
	  // The result of this call should be consistent with the result
	  // of the call to resolve_instance_class_or_null().
	  // See evaluation 6790209 and 4474172 for more details.
	  class_loader = Handle(THREAD, java_lang_ClassLoader::non_reflection_class_loader(class_loader()));
	
	  unsigned int d_hash = dictionary()->compute_hash(class_name, class_loader);
	  int d_index = dictionary()->hash_to_index(d_hash);
	
  {- -------------------------------------------
  (1) Dictionary::find() を呼んで
      引数で指定された klassOop を SystemDictionary 内から探し, 
      結果としてリターンする.
      ---------------------------------------- -}

	  {
	    // Note that we have an entry, and entries can be deleted only during GC,
	    // so we cannot allow GC to occur while we're holding this entry.
	    // We're using a No_Safepoint_Verifier to catch any place where we
	    // might potentially do a GC at all.
	    // SystemDictionary::do_unloading() asserts that classes are only
	    // unloaded at a safepoint.
	    No_Safepoint_Verifier nosafepoint;
	    return dictionary()->find(d_index, d_hash, class_name, class_loader,
	                              protection_domain, THREAD);
	  }
	}
	
```


