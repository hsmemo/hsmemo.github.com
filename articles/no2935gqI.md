---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
void VM_GetOrSetLocal::doit() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (この関数の処理は, _set フィールドの値に応じて ２通りに分岐.
       このフィールドの値はコンストラクタ引数で指定され, 局所変数を get するのか set するのかを示す)
      ---------------------------------------- -}

  {- -------------------------------------------
  (1) (以下は, 局所変数を set する場合の処理)
      ---------------------------------------- -}

	  if (_set) {
	    // Force deoptimization of frame if compiled because it's
	    // possible the compiler emitted some locals as constant values,
	    // meaning they are not mutable.
	    if (can_be_deoptimized(_jvf)) {
	
	      // Schedule deoptimization so that eventually the local
	      // update will be written to an interpreter frame.
	      Deoptimization::deoptimize_frame(_jvf->thread(), _jvf->fr().id());
	
	      // Now store a new value for the local which will be applied
	      // once deoptimization occurs. Note however that while this
	      // write is deferred until deoptimization actually happens
	      // can vframe created after this point will have its locals
	      // reflecting this update so as far as anyone can see the
	      // write has already taken place.
	
	      // If we are updating an oop then get the oop from the handle
	      // since the handle will be long gone by the time the deopt
	      // happens. The oop stored in the deferred local will be
	      // gc'd on its own.
	      if (_type == T_OBJECT) {
	        _value.l = (jobject) (JNIHandles::resolve_external_guard(_value.l));
	      }
	      // Re-read the vframe so we can see that it is deoptimized
	      // [ Only need because of assert in update_local() ]
	      _jvf = get_java_vframe();
	      ((compiledVFrame*)_jvf)->update_local(_type, _index, _value);
	      return;
	    }
	    StackValueCollection *locals = _jvf->locals();
	    HandleMark hm;
	
	    switch (_type) {
	      case T_INT:    locals->set_int_at   (_index, _value.i); break;
	      case T_LONG:   locals->set_long_at  (_index, _value.j); break;
	      case T_FLOAT:  locals->set_float_at (_index, _value.f); break;
	      case T_DOUBLE: locals->set_double_at(_index, _value.d); break;
	      case T_OBJECT: {
	        Handle ob_h(JNIHandles::resolve_external_guard(_value.l));
	        locals->set_obj_at (_index, ob_h);
	        break;
	      }
	      default: ShouldNotReachHere();
	    }
	    _jvf->set_locals(locals);

  {- -------------------------------------------
  (1) (以下は, 局所変数を get する場合の処理)
      ---------------------------------------- -}

	  } else {
	    if (_jvf->method()->is_native() && _jvf->is_compiled_frame()) {
	      assert(getting_receiver(), "Can only get here when getting receiver");
	      oop receiver = _jvf->fr().get_native_receiver();
	      _value.l = JNIHandles::make_local(_calling_thread, receiver);
	    } else {
	      StackValueCollection *locals = _jvf->locals();
	
	      if (locals->at(_index)->type() == T_CONFLICT) {
	        memset(&_value, 0, sizeof(_value));
	        _value.l = NULL;
	        return;
	      }
	
	      switch (_type) {
	        case T_INT:    _value.i = locals->int_at   (_index);   break;
	        case T_LONG:   _value.j = locals->long_at  (_index);   break;
	        case T_FLOAT:  _value.f = locals->float_at (_index);   break;
	        case T_DOUBLE: _value.d = locals->double_at(_index);   break;
	        case T_OBJECT: {
	          // Wrap the oop to be returned in a local JNI handle since
	          // oops_do() no longer applies after doit() is finished.
	          oop obj = locals->obj_at(_index)();
	          _value.l = JNIHandles::make_local(_calling_thread, obj);
	          break;
	        }
	        default: ShouldNotReachHere();
	      }
	    }
	  }
	}
	
```


