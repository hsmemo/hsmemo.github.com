---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiEnvBase.cpp
### 説明(description)

```
// Verifies that the top frame is a java frame in an expected state.
// Deoptimizes frame if needed.
// Checks that the frame method signature matches the return type (tos).
// HandleMark must be defined in the caller only.
// It is to keep a ret_ob_h handle alive after return to the caller.
```

### 名前(function name)
```
jvmtiError
JvmtiEnvBase::check_top_frame(JavaThread* current_thread, JavaThread* java_thread,
                              jvalue value, TosState tos, Handle* ret_ob_h) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ResourceMark rm(current_thread);
	
  {- -------------------------------------------
  (1) 対象のスレッドのスタックの先頭にあるフレームを取得する.
      (もしフレームがなければ, ここでリターン(JVMTI_ERROR_NO_MORE_FRAMES).
       これは JVMTI 仕様で要求されている挙動.)
      ---------------------------------------- -}

	  vframe *vf = vframeFor(java_thread, 0);
	  NULL_CHECK(vf, JVMTI_ERROR_NO_MORE_FRAMES);
	
  {- -------------------------------------------
  (1) 取得した先頭フレームが non-native な Java メソッドのフレームでなければ, ここでリターン(JVMTI_ERROR_OPAQUE_FRAME).
      (これは JVMTI 仕様で要求されている挙動)
      ---------------------------------------- -}

	  javaVFrame *jvf = (javaVFrame*) vf;
	  if (!vf->is_java_frame() || jvf->method()->is_native()) {
	    return JVMTI_ERROR_OPAQUE_FRAME;
	  }
	
  {- -------------------------------------------
  (1) 先頭フレームが compiled frame だった場合は, Deoptimization::deoptimize_frame() で deopt しておく.
      ---------------------------------------- -}

	  // If the frame is a compiled one, need to deoptimize it.
	  if (vf->is_compiled_frame()) {
	    if (!vf->fr().can_be_deoptimized()) {
	      return JVMTI_ERROR_OPAQUE_FRAME;
	    }
	    Deoptimization::deoptimize_frame(java_thread, jvf->fr().id());
	  }
	
  {- -------------------------------------------
  (1) メソッドの返り値の型を取得し, 使用された ForceEarlyReturn*() の種別(メソッド名の '*' の部分) と型があっているかどうかを確認する.
      (例えば, 返り値が int なら ForceEarlyReturnInt() が使われているかどうか)
  
      あっていなければ, ここでリターン(JVMTI_ERROR_TYPE_MISMATCH).
      ---------------------------------------- -}

	  // Get information about method return type
	  Symbol* signature = jvf->method()->signature();
	
	  ResultTypeFinder rtf(signature);
	  TosState fr_tos = as_TosState(rtf.type());
	  if (fr_tos != tos) {
	    if (tos != itos || (fr_tos != btos && fr_tos != ctos && fr_tos != stos)) {
	      return JVMTI_ERROR_TYPE_MISMATCH;
	    }
	  }
	
  {- -------------------------------------------
  (1) さらに, メソッドの返り値の型が reference (Object 型)である場合には,
      返り値として正しいオブジェクトが指定されているかどうかをチェックしておく.
  
      以下の場合はここでリターン.
      * オブジェクトとしておかしい場合(JVMTI_ERROR_INVALID_OBJECT).
      * 返り値のクラス(またはそのサブクラス)でない場合(JVMTI_ERROR_TYPE_MISMATCH).
      ---------------------------------------- -}

	  // Check that the jobject class matches the return type signature.
	  jobject jobj = value.l;
	  if (tos == atos && jobj != NULL) { // NULL reference is allowed
	    Handle ob_h = Handle(current_thread, JNIHandles::resolve_external_guard(jobj));
	    NULL_CHECK(ob_h, JVMTI_ERROR_INVALID_OBJECT);
	    KlassHandle ob_kh = KlassHandle(current_thread, ob_h()->klass());
	    NULL_CHECK(ob_kh, JVMTI_ERROR_INVALID_OBJECT);
	
	    // Method return type signature.
	    char* ty_sign = 1 + strchr(signature->as_C_string(), ')');
	
	    if (!VM_GetOrSetLocal::is_assignable(ty_sign, Klass::cast(ob_kh()), current_thread)) {
	      return JVMTI_ERROR_TYPE_MISMATCH;
	    }
	    *ret_ob_h = ob_h;
	  }

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	} /* end check_top_frame */
	
```


