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
JNI_ENTRY(jclass, jni_GetSuperclass(JNIEnv *env, jclass sub))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("GetSuperclass");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE2(hotspot_jni, GetSuperclass__entry, env, sub);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jclass obj = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(GetSuperclass, jclass, (const jclass&)obj);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  oop mirror = JNIHandles::resolve_non_null(sub);

  {- -------------------------------------------
  (1) プリミティブ型の場合には, NULL をリターン.
      ---------------------------------------- -}

	  // primitive classes return NULL
	  if (java_lang_Class::is_primitive(mirror)) return NULL;
	
  {- -------------------------------------------
  (1) 対象の種類に応じて, 以下の値をリターンする.
      * インターフェースの場合 (= is_interface() が true の場合):
        NULL をリターン
      * それ以外の場合:
        Klass::java_super() の結果をリターン.
        (配列クラスの場合は java.lang.Object が, それ以外の場合はそのスーパークラスが返される)
      ---------------------------------------- -}

	  // Rules of Class.getSuperClass as implemented by KLass::java_super:
	  // arrays return Object
	  // interfaces return NULL
	  // proper classes return Klass::super()
	  klassOop k = java_lang_Class::as_klassOop(mirror);
	  if (Klass::cast(k)->is_interface()) return NULL;
	
	  // return mirror for superclass
	  klassOop super = Klass::cast(k)->java_super();
	  // super2 is the value computed by the compiler's getSuperClass intrinsic:
	  debug_only(klassOop super2 = ( Klass::cast(k)->oop_is_javaArray()
	                                 ? SystemDictionary::Object_klass()
	                                 : Klass::cast(k)->super() ) );
	  assert(super == super2,
	         "java_super computation depends on interface, array, other super");
	  obj = (super == NULL) ? NULL : (jclass) JNIHandles::make_local(Klass::cast(super)->java_mirror());
	  return obj;
	JNI_END
	
```


