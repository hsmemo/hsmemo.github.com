---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp
### 説明(description)

```
// Checks error conditions:
//   JVMTI_ERROR_INVALID_SLOT
//   JVMTI_ERROR_TYPE_MISMATCH
// Returns: 'true' - everything is Ok, 'false' - error code

```

### 名前(function name)
```
bool VM_GetOrSetLocal::check_slot_type(javaVFrame* jvf) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodOop method_oop = jvf->method();

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  if (!method_oop->has_localvariable_table()) {
	    // Just to check index boundaries
	    jint extra_slot = (_type == T_LONG || _type == T_DOUBLE) ? 1 : 0;
	    if (_index < 0 || _index + extra_slot >= method_oop->max_locals()) {
	      _result = JVMTI_ERROR_INVALID_SLOT;
	      return false;
	    }
	    return true;
	  }
	
  {- -------------------------------------------
  (1) もし局所変数が1つもなければ, false をリターン.
      ---------------------------------------- -}

	  jint num_entries = method_oop->localvariable_table_length();
	  if (num_entries == 0) {
	    _result = JVMTI_ERROR_INVALID_SLOT;
	    return false;       // There are no slots
	  }

  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  int signature_idx = -1;
	  int vf_bci = jvf->bci();
	  LocalVariableTableElement* table = method_oop->localvariable_table_start();
	  for (int i = 0; i < num_entries; i++) {
	    int start_bci = table[i].start_bci;
	    int end_bci = start_bci + table[i].length;
	
	    // Here we assume that locations of LVT entries
	    // with the same slot number cannot be overlapped
	    if (_index == (jint) table[i].slot && start_bci <= vf_bci && vf_bci <= end_bci) {
	      signature_idx = (int) table[i].descriptor_cp_index;
	      break;
	    }
	  }
	  if (signature_idx == -1) {
	    _result = JVMTI_ERROR_INVALID_SLOT;
	    return false;       // Incorrect slot index
	  }
	  Symbol*   sign_sym  = method_oop->constants()->symbol_at(signature_idx);
	  const char* signature = (const char *) sign_sym->as_utf8();
	  BasicType slot_type = char2type(signature[0]);
	
	  switch (slot_type) {
	  case T_BYTE:
	  case T_SHORT:
	  case T_CHAR:
	  case T_BOOLEAN:
	    slot_type = T_INT;
	    break;
	  case T_ARRAY:
	    slot_type = T_OBJECT;
	    break;
	  };
	  if (_type != slot_type) {
	    _result = JVMTI_ERROR_TYPE_MISMATCH;
	    return false;
	  }
	
	  jobject jobj = _value.l;
	  if (_set && slot_type == T_OBJECT && jobj != NULL) { // NULL reference is allowed
	    // Check that the jobject class matches the return type signature.
	    JavaThread* cur_thread = JavaThread::current();
	    HandleMark hm(cur_thread);
	
	    Handle obj = Handle(cur_thread, JNIHandles::resolve_external_guard(jobj));
	    NULL_CHECK(obj, (_result = JVMTI_ERROR_INVALID_OBJECT, false));
	    KlassHandle ob_kh = KlassHandle(cur_thread, obj->klass());
	    NULL_CHECK(ob_kh, (_result = JVMTI_ERROR_INVALID_OBJECT, false));
	
	    if (!is_assignable(signature, Klass::cast(ob_kh()), cur_thread)) {
	      _result = JVMTI_ERROR_TYPE_MISMATCH;
	      return false;
	    }
	  }
	  return true;
	}
	
```


