---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiRedefineClasses.cpp

### 名前(function name)
```
void VM_RedefineClasses::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Thread *thread = Thread::current();
	
  {- -------------------------------------------
  (1) UseSharedSpaces オプションが指定されている場合は, (このままだとクラスを再定義できないので)
      CompactingPermGenGen::remap_shared_readonly_as_readwrite() を読んで
      shared readonly space を shared readwrite に変更しておく.
      (なお, 失敗したらここでリターン(JVMTI_ERROR_INTERNAL))
      ---------------------------------------- -}

	  if (UseSharedSpaces) {
	    // Sharing is enabled so we remap the shared readonly space to
	    // shared readwrite, private just in case we need to redefine
	    // a shared class. We do the remap during the doit() phase of
	    // the safepoint to be safer.
	    if (!CompactingPermGenGen::remap_shared_readonly_as_readwrite()) {
	      RC_TRACE_WITH_THREAD(0x00000001, thread,
	        ("failed to remap shared readonly space to readwrite, private"));
	      _res = JVMTI_ERROR_INTERNAL;
	      return;
	    }
	  }
	
  {- -------------------------------------------
  (1) 処理対象の全てのクラスに対して, 
      VM_RedefineClasses::redefine_single_class() を呼び出してクラスを再定義する.
      ---------------------------------------- -}

	  for (int i = 0; i < _class_count; i++) {
	    redefine_single_class(_class_defs[i].klass, _scratch_classes[i], thread);
	  }

  {- -------------------------------------------
  (1) SystemDictionary::notice_modification() で変更回数をあげておく.
      (コンパイル途中にメソッドが変更された場合に, 不正になってしまったコンパイル結果を破棄するために必要な処理.
       (See: ciEnv::system_dictionary_modification_counter_changed()))
      ---------------------------------------- -}

	  // Disable any dependent concurrent compilations
	  SystemDictionary::notice_modification();
	
  {- -------------------------------------------
  (1) JvmtiExport::_has_redefined_a_class フィールドを true にしておく. #TODO
      ---------------------------------------- -}

	  // Set flag indicating that some invariants are no longer true.
	  // See jvmtiExport.hpp for detailed explanation.
	  JvmtiExport::set_has_redefined_a_class();
	
  {- -------------------------------------------
  (1) (デバッグ用の処理) (#ifdef ASSERT 時にのみ実行)
      ---------------------------------------- -}

	#ifdef ASSERT
	  SystemDictionary::classes_do(check_class, thread);
	#endif
	}
	
```


