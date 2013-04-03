---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/classFileParser.cpp

### 名前(function name)
```
constantPoolHandle ClassFileParser::parse_constant_pool(TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ClassFileStream* cfs = stream();
	  constantPoolHandle nullHandle;
	
  {- -------------------------------------------
  (1) まず, constant_pool_count 情報(長さ情報) を読み込む
      ---------------------------------------- -}

	  cfs->guarantee_more(3, CHECK_(nullHandle)); // length, first cp tag
	  u2 length = cfs->get_u2_fast();
	  guarantee_property(
	    length >= 1, "Illegal constant pool size %u in class file %s",
	    length, CHECK_(nullHandle));

  {- -------------------------------------------
  (1) 長さ分だけの constantPoolOop を確保し, 初期化を行う (ついでに Handle を作っておく).
      ---------------------------------------- -}

	  constantPoolOop constant_pool =
	                      oopFactory::new_constantPool(length,
	                                                   oopDesc::IsSafeConc,
	                                                   CHECK_(nullHandle));
	  constantPoolHandle cp (THREAD, constant_pool);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  cp->set_partially_loaded();    // Enables heap verify to work on partial constantPoolOops

  {- -------------------------------------------
  (1) (パース処理でエラーが起こった場合に, 
       インクリメントした Symbol の参照カウンタ(reference count)が元に戻るようにしておく)
      (See: ConstantPoolCleaner)
      ---------------------------------------- -}

	  ConstantPoolCleaner cp_in_error(cp); // set constant pool to be cleaned up.
	
  {- -------------------------------------------
  (1) ClassFileParser::parse_constant_pool_entries() を呼んで, constant pool 内の各エントリを読み込む
      ---------------------------------------- -}

	  // parsing constant pool entries
	  parse_constant_pool_entries(cp, length, CHECK_(nullHandle));
	
  {- -------------------------------------------
  (1) 内容の vefify と String や Class 部分の Unresolved タグへの書き換えを行う
      (verify 部分は JVMS 4.10 に書かれている下2つの処理に相当? 
       "The verifier also performs verification that can be done 
       without looking at the code array of the Code attribute" 以下に書かれている部分.)
    
      verify は2段階で行っている模様. (ただし, _need_verify が立っていなければ 1段階目で終了).
      ここではまず1段階目の verify として, 以下のようなチェックを行う.
      * 他の constant pool entry を参照する entry (= constant pool index を保持するような entry) の場合に, 
        正しい種別を指しているか?, 
      * Long や Double 定数の場合, (2 word 分の領域を使うので) constant pool 領域からはみ出していないか, 
        また 2word 目は tags 上では invalid になっているか？
  
      また, JVM_CONSTANT_StringIndex や JVM_CONSTANT_ClassIndex のものについては, 
      チェック終了後に tags を JVM_CONSTANT_UnresolvedClass や JVM_CONSTANT_UnresolvedString に置き換えている.
      (また, タグだけでなく値も, constant pool に書かれていた情報そのものではなく, Symbol 情報だけに変更している)
      ---------------------------------------- -}

	  int index = 1;  // declared outside of loops for portability
	
	  // first verification pass - validate cross references and fixup class and string constants
	  for (index = 1; index < length; index++) {          // Index 0 is unused
	    jbyte tag = cp->tag_at(index).value();
	    switch (tag) {
	      case JVM_CONSTANT_Class :
	        ShouldNotReachHere();     // Only JVM_CONSTANT_ClassIndex should be present
	        break;
	      case JVM_CONSTANT_Fieldref :
	        // fall through
	      case JVM_CONSTANT_Methodref :
	        // fall through
	      case JVM_CONSTANT_InterfaceMethodref : {
	        if (!_need_verify) break;
	        int klass_ref_index = cp->klass_ref_index_at(index);
	        int name_and_type_ref_index = cp->name_and_type_ref_index_at(index);
	        check_property(valid_cp_range(klass_ref_index, length) &&
	                       is_klass_reference(cp, klass_ref_index),
	                       "Invalid constant pool index %u in class file %s",
	                       klass_ref_index,
	                       CHECK_(nullHandle));
	        check_property(valid_cp_range(name_and_type_ref_index, length) &&
	                       cp->tag_at(name_and_type_ref_index).is_name_and_type(),
	                       "Invalid constant pool index %u in class file %s",
	                       name_and_type_ref_index,
	                       CHECK_(nullHandle));
	        break;
	      }
	      case JVM_CONSTANT_String :
	        ShouldNotReachHere();     // Only JVM_CONSTANT_StringIndex should be present
	        break;
	      case JVM_CONSTANT_Integer :
	        break;
	      case JVM_CONSTANT_Float :
	        break;
	      case JVM_CONSTANT_Long :
	      case JVM_CONSTANT_Double :
	        index++;
	        check_property(
	          (index < length && cp->tag_at(index).is_invalid()),
	          "Improper constant pool long/double index %u in class file %s",
	          index, CHECK_(nullHandle));
	        break;
	      case JVM_CONSTANT_NameAndType : {
	        if (!_need_verify) break;
	        int name_ref_index = cp->name_ref_index_at(index);
	        int signature_ref_index = cp->signature_ref_index_at(index);
	        check_property(
	          valid_cp_range(name_ref_index, length) &&
	            cp->tag_at(name_ref_index).is_utf8(),
	          "Invalid constant pool index %u in class file %s",
	          name_ref_index, CHECK_(nullHandle));
	        check_property(
	          valid_cp_range(signature_ref_index, length) &&
	            cp->tag_at(signature_ref_index).is_utf8(),
	          "Invalid constant pool index %u in class file %s",
	          signature_ref_index, CHECK_(nullHandle));
	        break;
	      }
	      case JVM_CONSTANT_Utf8 :
	        break;
	      case JVM_CONSTANT_UnresolvedClass :         // fall-through
	      case JVM_CONSTANT_UnresolvedClassInError:
	        ShouldNotReachHere();     // Only JVM_CONSTANT_ClassIndex should be present
	        break;
	      case JVM_CONSTANT_ClassIndex :
	        {
	          int class_index = cp->klass_index_at(index);
	          check_property(
	            valid_cp_range(class_index, length) &&
	              cp->tag_at(class_index).is_utf8(),
	            "Invalid constant pool index %u in class file %s",
	            class_index, CHECK_(nullHandle));
	          cp->unresolved_klass_at_put(index, cp->symbol_at(class_index));
	        }
	        break;
	      case JVM_CONSTANT_UnresolvedString :
	        ShouldNotReachHere();     // Only JVM_CONSTANT_StringIndex should be present
	        break;
	      case JVM_CONSTANT_StringIndex :
	        {
	          int string_index = cp->string_index_at(index);
	          check_property(
	            valid_cp_range(string_index, length) &&
	              cp->tag_at(string_index).is_utf8(),
	            "Invalid constant pool index %u in class file %s",
	            string_index, CHECK_(nullHandle));
	          Symbol* sym = cp->symbol_at(string_index);
	          cp->unresolved_string_at_put(index, sym);
	        }
	        break;
	      case JVM_CONSTANT_MethodHandle :
	        {
	          int ref_index = cp->method_handle_index_at(index);
	          check_property(
	            valid_cp_range(ref_index, length) &&
	                EnableInvokeDynamic,
	              "Invalid constant pool index %u in class file %s",
	              ref_index, CHECK_(nullHandle));
	          constantTag tag = cp->tag_at(ref_index);
	          int ref_kind  = cp->method_handle_ref_kind_at(index);
	          switch (ref_kind) {
	          case JVM_REF_getField:
	          case JVM_REF_getStatic:
	          case JVM_REF_putField:
	          case JVM_REF_putStatic:
	            check_property(
	              tag.is_field(),
	              "Invalid constant pool index %u in class file %s (not a field)",
	              ref_index, CHECK_(nullHandle));
	            break;
	          case JVM_REF_invokeVirtual:
	          case JVM_REF_invokeStatic:
	          case JVM_REF_invokeSpecial:
	          case JVM_REF_newInvokeSpecial:
	            check_property(
	              tag.is_method(),
	              "Invalid constant pool index %u in class file %s (not a method)",
	              ref_index, CHECK_(nullHandle));
	            break;
	          case JVM_REF_invokeInterface:
	            check_property(
	              tag.is_interface_method(),
	              "Invalid constant pool index %u in class file %s (not an interface method)",
	              ref_index, CHECK_(nullHandle));
	            break;
	          default:
	            classfile_parse_error(
	              "Bad method handle kind at constant pool index %u in class file %s",
	              index, CHECK_(nullHandle));
	          }
	          // Keep the ref_index unchanged.  It will be indirected at link-time.
	        }
	        break;
	      case JVM_CONSTANT_MethodType :
	        {
	          int ref_index = cp->method_type_index_at(index);
	          check_property(
	            valid_cp_range(ref_index, length) &&
	                cp->tag_at(ref_index).is_utf8() &&
	                EnableInvokeDynamic,
	              "Invalid constant pool index %u in class file %s",
	              ref_index, CHECK_(nullHandle));
	        }
	        break;
	      case JVM_CONSTANT_InvokeDynamic :
	        {
	          int name_and_type_ref_index = cp->invoke_dynamic_name_and_type_ref_index_at(index);
	          check_property(valid_cp_range(name_and_type_ref_index, length) &&
	                         cp->tag_at(name_and_type_ref_index).is_name_and_type(),
	                         "Invalid constant pool index %u in class file %s",
	                         name_and_type_ref_index,
	                         CHECK_(nullHandle));
	          // bootstrap specifier index must be checked later, when BootstrapMethods attr is available
	          break;
	        }
	      default:
	        fatal(err_msg("bad constant pool tag value %u",
	                      cp->tag_at(index).value()));
	        ShouldNotReachHere();
	        break;
	    } // end of switch
	  } // end of for
	
  {- -------------------------------------------
  (1) ?? (#TODO)
      (普通のロード処理では関係ない？？
       引数の _cp_patches は, Unsafe_DefineAnonymousClass_impl() から呼ばれたときしか設定されてない模様).
      ---------------------------------------- -}

	  if (_cp_patches != NULL) {
	    // need to treat this_class specially...
	    assert(EnableInvokeDynamic, "");
	    int this_class_index;
	    {
	      cfs->guarantee_more(8, CHECK_(nullHandle));  // flags, this_class, super_class, infs_len
	      u1* mark = cfs->current();
	      u2 flags         = cfs->get_u2_fast();
	      this_class_index = cfs->get_u2_fast();
	      cfs->set_current(mark);  // revert to mark
	    }
	
	    for (index = 1; index < length; index++) {          // Index 0 is unused
	      if (has_cp_patch_at(index)) {
	        guarantee_property(index != this_class_index,
	                           "Illegal constant pool patch to self at %d in class file %s",
	                           index, CHECK_(nullHandle));
	        patch_constant_pool(cp, index, cp_patch_at(index), CHECK_(nullHandle));
	      }
	    }
	    // Ensure that all the patches have been used.
	    for (index = 0; index < _cp_patches->length(); index++) {
	      guarantee_property(!has_cp_patch_at(index),
	                         "Unused constant pool patch at %d in class file %s",
	                         index, CHECK_(nullHandle));
	    }
	  }
	
  {- -------------------------------------------
  (1) verify が必要なければここで終了 
      (ConstantPoolCleaner オブジェクトの _in_error をクリアし, パースした結果をリターン)
      ---------------------------------------- -}

	  if (!_need_verify) {
	    cp_in_error.set_in_error(false);
	    return cp;
	  }
	
  {- -------------------------------------------
  (1) 2段階目の verify 処理を行う.
      ---------------------------------------- -}

	  // second verification pass - checks the strings are of the right format.
	  // but not yet to the other entries
	  for (index = 1; index < length; index++) {
	    jbyte tag = cp->tag_at(index).value();
	    switch (tag) {
	      case JVM_CONSTANT_UnresolvedClass: {
	        Symbol*  class_name = cp->unresolved_klass_at(index);
	        // check the name, even if _cp_patches will overwrite it
	        verify_legal_class_name(class_name, CHECK_(nullHandle));
	        break;
	      }
	      case JVM_CONSTANT_NameAndType: {
	        if (_need_verify && _major_version >= JAVA_7_VERSION) {
	          int sig_index = cp->signature_ref_index_at(index);
	          int name_index = cp->name_ref_index_at(index);
	          Symbol*  name = cp->symbol_at(name_index);
	          Symbol*  sig = cp->symbol_at(sig_index);
	          if (sig->byte_at(0) == JVM_SIGNATURE_FUNC) {
	            verify_legal_method_signature(name, sig, CHECK_(nullHandle));
	          } else {
	            verify_legal_field_signature(name, sig, CHECK_(nullHandle));
	          }
	        }
	        break;
	      }
	      case JVM_CONSTANT_InvokeDynamic:
	      case JVM_CONSTANT_Fieldref:
	      case JVM_CONSTANT_Methodref:
	      case JVM_CONSTANT_InterfaceMethodref: {
	        int name_and_type_ref_index = cp->name_and_type_ref_index_at(index);
	        // already verified to be utf8
	        int name_ref_index = cp->name_ref_index_at(name_and_type_ref_index);
	        // already verified to be utf8
	        int signature_ref_index = cp->signature_ref_index_at(name_and_type_ref_index);
	        Symbol*  name = cp->symbol_at(name_ref_index);
	        Symbol*  signature = cp->symbol_at(signature_ref_index);
	        if (tag == JVM_CONSTANT_Fieldref) {
	          verify_legal_field_name(name, CHECK_(nullHandle));
	          if (_need_verify && _major_version >= JAVA_7_VERSION) {
	            // Signature is verified above, when iterating NameAndType_info.
	            // Need only to be sure it's the right type.
	            if (signature->byte_at(0) == JVM_SIGNATURE_FUNC) {
	              throwIllegalSignature(
	                  "Field", name, signature, CHECK_(nullHandle));
	            }
	          } else {
	            verify_legal_field_signature(name, signature, CHECK_(nullHandle));
	          }
	        } else {
	          verify_legal_method_name(name, CHECK_(nullHandle));
	          if (_need_verify && _major_version >= JAVA_7_VERSION) {
	            // Signature is verified above, when iterating NameAndType_info.
	            // Need only to be sure it's the right type.
	            if (signature->byte_at(0) != JVM_SIGNATURE_FUNC) {
	              throwIllegalSignature(
	                  "Method", name, signature, CHECK_(nullHandle));
	            }
	          } else {
	            verify_legal_method_signature(name, signature, CHECK_(nullHandle));
	          }
	          if (tag == JVM_CONSTANT_Methodref) {
	            // 4509014: If a class method name begins with '<', it must be "<init>".
	            assert(name != NULL, "method name in constant pool is null");
	            unsigned int name_len = name->utf8_length();
	            assert(name_len > 0, "bad method name");  // already verified as legal name
	            if (name->byte_at(0) == '<') {
	              if (name != vmSymbols::object_initializer_name()) {
	                classfile_parse_error(
	                  "Bad method name at constant pool index %u in class file %s",
	                  name_ref_index, CHECK_(nullHandle));
	              }
	            }
	          }
	        }
	        break;
	      }
	      case JVM_CONSTANT_MethodHandle: {
	        int ref_index = cp->method_handle_index_at(index);
	        int ref_kind  = cp->method_handle_ref_kind_at(index);
	        switch (ref_kind) {
	        case JVM_REF_invokeVirtual:
	        case JVM_REF_invokeStatic:
	        case JVM_REF_invokeSpecial:
	        case JVM_REF_newInvokeSpecial:
	          {
	            int name_and_type_ref_index = cp->name_and_type_ref_index_at(ref_index);
	            int name_ref_index = cp->name_ref_index_at(name_and_type_ref_index);
	            Symbol*  name = cp->symbol_at(name_ref_index);
	            if (ref_kind == JVM_REF_newInvokeSpecial) {
	              if (name != vmSymbols::object_initializer_name()) {
	                classfile_parse_error(
	                  "Bad constructor name at constant pool index %u in class file %s",
	                  name_ref_index, CHECK_(nullHandle));
	              }
	            } else {
	              if (name == vmSymbols::object_initializer_name()) {
	                classfile_parse_error(
	                  "Bad method name at constant pool index %u in class file %s",
	                  name_ref_index, CHECK_(nullHandle));
	              }
	            }
	          }
	          break;
	          // Other ref_kinds are already fully checked in previous pass.
	        }
	        break;
	      }
	      case JVM_CONSTANT_MethodType: {
	        Symbol* no_name = vmSymbols::type_name(); // place holder
	        Symbol*  signature = cp->method_type_signature_at(index);
	        verify_legal_method_signature(no_name, signature, CHECK_(nullHandle));
	        break;
	      }
	      case JVM_CONSTANT_Utf8: {
	        assert(cp->symbol_at(index)->refcount() != 0, "count corrupted");
	      }
	    }  // end of switch
	  }  // end of for
	
  {- -------------------------------------------
  (1) 終了
      (ConstantPoolCleaner オブジェクトの _in_error をクリアし, パースした結果をリターン)
      ---------------------------------------- -}

	  cp_in_error.set_in_error(false);
	  return cp;
	
```


