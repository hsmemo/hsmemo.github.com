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
JNI_ENTRY(jfieldID, jni_GetStaticFieldID(JNIEnv *env, jclass clazz,
          const char *name, const char *sig))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (デバッグ用の処理)
      (See: JNIWrapper)
      ---------------------------------------- -}

	  JNIWrapper("GetStaticFieldID");

  {- -------------------------------------------
  (1) (DTrace のフック点)
      ---------------------------------------- -}

	  DTRACE_PROBE4(hotspot_jni, GetStaticFieldID__entry, env, clazz, name, sig);

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jfieldID ret = NULL;

  {- -------------------------------------------
  (1) (DTrace のフック点)(リターン用) (See: DT_RETURN_MARK)
      ---------------------------------------- -}

	  DT_RETURN_MARK(GetStaticFieldID, jfieldID, (const jfieldID&)ret);
	
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // The class should have been loaded (we have an instance of the class
	  // passed in) so the field and signature should already be in the symbol
	  // table.  If they're not there, the field doesn't exist.
	  TempNewSymbol fieldname = SymbolTable::probe(name, (int)strlen(name));
	  TempNewSymbol signame = SymbolTable::probe(sig, (int)strlen(sig));

  {- -------------------------------------------
  (1) もし指定されたフィールド名やシグネチャが SynbolTable 内に見つからなければ, NoSuchMethodError.
      ---------------------------------------- -}

	  if (fieldname == NULL || signame == NULL) {
	    THROW_MSG_0(vmSymbols::java_lang_NoSuchFieldError(), (char*) name);
	  }

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  KlassHandle k(THREAD,
	                java_lang_Class::as_klassOop(JNIHandles::resolve_non_null(clazz)));

  {- -------------------------------------------
  (1) 対象のクラスオブジェクトに対して Klass::initialize() を呼び, 
      リンクや初期化が終わっていなければ終わらせておく.
      ---------------------------------------- -}

	  // Make sure class is initialized before handing id's out to static fields
	  Klass::cast(k())->initialize(CHECK_NULL);
	
  {- -------------------------------------------
  (1) instanceKlass::find_field() を呼んで, 指定されたフィールドの情報を取得する.
  
      (instanceKlass::find_field() で見つからなかった場合や
       そもそも instanceKlass ではなかった場合は, NoSuchFieldError.)
      ---------------------------------------- -}

	  fieldDescriptor fd;
	  if (!Klass::cast(k())->oop_is_instance() ||
	      !instanceKlass::cast(k())->find_field(fieldname, signame, true, &fd)) {
	    THROW_MSG_0(vmSymbols::java_lang_NoSuchFieldError(), (char*) name);
	  }
	
  {- -------------------------------------------
  (1) instanceKlass::jni_id_for() で対応する JNIid 値を取得し, 
      それを jfieldIDWorkaround::to_static_jfieldID() で jfieldID 化したものをリターン.
      ---------------------------------------- -}

	  // A jfieldID for a static field is a JNIid specifying the field holder and the offset within the klassOop
	  JNIid* id = instanceKlass::cast(fd.field_holder())->jni_id_for(fd.offset());
	  debug_only(id->set_is_static_field_id();)
	
	  debug_only(id->verify(fd.field_holder()));
	
	  ret = jfieldIDWorkaround::to_static_jfieldID(id);
	  return ret;
	JNI_END
	
```


