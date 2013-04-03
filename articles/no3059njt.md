---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jni.cpp

### 名前(function name)
```
static void jni_invoke_nonstatic(JNIEnv *env, JavaValue* result, jobject receiver, JNICallType call_type, jmethodID method_id, JNI_ArgumentPusher *args, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) もし引数で指定された receiver が NULL だったら. NullPointerException.
      ---------------------------------------- -}

	  oop recv = JNIHandles::resolve(receiver);
	  if (recv == NULL) {
	    THROW(vmSymbols::java_lang_NullPointerException());
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  Handle h_recv(THREAD, recv);
	
	  int number_of_parameters;
	  methodOop selected_method;

  {- -------------------------------------------
  (1) 以下のブロック内で dynamic dispatch 処理を行い, 実際に呼び出す methodOop を探し当てる.
      ---------------------------------------- -}

	  {
	    methodOop m = JNIHandles::resolve_jmethod_id(method_id);
	    number_of_parameters = m->size_of_parameters();
	    klassOop holder = m->method_holder();
	    if (!(Klass::cast(holder))->is_interface()) {
	      // non-interface call -- for that little speed boost, don't handlize
	      debug_only(No_Safepoint_Verifier nosafepoint;)
	      if (call_type == JNI_VIRTUAL) {
	        // jni_GetMethodID makes sure class is linked and initialized
	        // so m should have a valid vtable index.
	        int vtbl_index = m->vtable_index();
	        if (vtbl_index != methodOopDesc::nonvirtual_vtable_index) {
	          klassOop k = h_recv->klass();
	          // k might be an arrayKlassOop but all vtables start at
	          // the same place. The cast is to avoid virtual call and assertion.
	          instanceKlass *ik = (instanceKlass*)k->klass_part();
	          selected_method = ik->method_at_vtable(vtbl_index);
	        } else {
	          // final method
	          selected_method = m;
	        }
	      } else {
	        // JNI_NONVIRTUAL call
	        selected_method = m;
	      }
	    } else {
	      // interface call
	      KlassHandle h_holder(THREAD, holder);
	
	      int itbl_index = m->cached_itable_index();
	      if (itbl_index == -1) {
	        itbl_index = klassItable::compute_itable_index(m);
	        m->set_cached_itable_index(itbl_index);
	        // the above may have grabbed a lock, 'm' and anything non-handlized can't be used again
	      }
	      klassOop k = h_recv->klass();
	      selected_method = instanceKlass::cast(k)->method_at_itable(h_holder(), itbl_index, CHECK);
	    }
	  }
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  methodHandle method(THREAD, selected_method);
	
  {- -------------------------------------------
  (1) (JavaCalls::call() では引数を JavaCallArguments オブジェクトに入れて渡す必要があるので)
      JavaCallArguments オブジェクトを用意し, 
      引数で渡された JNI_ArgumentPusher オブジェクト(args)にセットしておく.
      ---------------------------------------- -}

	  // Create object to hold arguments for the JavaCall, and associate it with
	  // the jni parser
	  ResourceMark rm(THREAD);
	  JavaCallArguments java_args(number_of_parameters);
	  args->set_java_argument_object(&java_args);
	
  {- -------------------------------------------
  (1) #TODO
      ---------------------------------------- -}

	  // handle arguments
	  assert(!method->is_static(), "method should not be static");
	  args->push_receiver(h_recv); // Push jobject handle
	
  {- -------------------------------------------
  (1) JNI_ArgumentPusher::iterate() を呼んで, 
      JavaCallArguments オブジェクト内に引数を詰め直す.
      ---------------------------------------- -}

	  // Fill out JavaCallArguments object
	  args->iterate( Fingerprinter(method).fingerprint() );

  {- -------------------------------------------
  (1) JavaValue::set_type() を呼んで, 
      引数で渡された JavaValue (result) に返値の型をセットしておく
      ---------------------------------------- -}

	  // Initialize result type
	  result->set_type(args->get_ret_type());
	
  {- -------------------------------------------
  (1) JavaCalls::call() によって, 引数で指定されたメソッドを呼び出す.
      ---------------------------------------- -}

	  // Invoke the method. Result is returned as oop.
	  JavaCalls::call(result, method, &java_args, CHECK);
	
  {- -------------------------------------------
  (1) 返値がポインタ(オブジェクト or 配列)の場合には, JNI Handle 化しておく.
      ---------------------------------------- -}

	  // Convert result
	  if (result->get_type() == T_OBJECT || result->get_type() == T_ARRAY) {
	    result->set_jobject(JNIHandles::make_local(env, (oop) result->get_jobject()));
	  }
	}
	
```


