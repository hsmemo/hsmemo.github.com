---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.cpp

### 名前(function name)
```
jvmtiError
JvmtiGetLoadedClasses::getClassLoaderClasses(JvmtiEnv *env, jobject initiatingLoader,
                                             jint* classCountPtr, jclass** classesPtr) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (SystemDictionary::classes_do() は Closure ではなく関数ポインタしか渡せないので, 
       クロージャー自体ではなくそれを使用する static メソッドを渡すことにしている)
      ---------------------------------------- -}

	  // Since SystemDictionary::classes_do only takes a function pointer
	  // and doesn't call back with a closure data pointer,
	  // we can only pass static methods.

  {- -------------------------------------------
  (1) (変数宣言など)
      (なお, コンストラクタ内でこの JvmtiGetLoadedClassesClosure オブジェクトを
       カレントスレッド内に格納する処理が行われる.
       これは SystemDictionary::classes_do() には自由変数的な引数が渡せないための措置)
      ---------------------------------------- -}

	  JvmtiGetLoadedClassesClosure closure(initiatingLoader);
	  {

  {- -------------------------------------------
  (1) (中途半端な結果にならないよう, 取得作業中はロックを確保しておく)
      ---------------------------------------- -}

	    // To get a consistent list of classes we need MultiArray_lock to ensure
	    // array classes aren't created, and SystemDictionary_lock to ensure that
	    // classes aren't added to the system dictionary,
	    MutexLocker ma(MultiArray_lock);
	    MutexLocker sd(SystemDictionary_lock);

  {- -------------------------------------------
  (1) まず該当するクラスの数を計算する.
      (この処理は, 全てのクラスに対して JvmtiGetLoadedClassesClosure::increment_with_loader() や
       JvmtiGetLoadedClassesClosure::increment_for_basic_type_arrays() を呼び出すことで行う)
      ---------------------------------------- -}

	    // First, count the classes in the system dictionary which have this loader recorded
	    // as an initiating loader. For basic type arrays this information is not recorded
	    // so GetClassLoaderClasses will return all of the basic type arrays. This is okay
	    // because the defining loader for basic type arrays is always the boot class loader
	    // and these classes are "visible" to all loaders.
	    SystemDictionary::classes_do(&JvmtiGetLoadedClassesClosure::increment_with_loader);
	    Universe::basic_type_classes_do(&JvmtiGetLoadedClassesClosure::increment_for_basic_type_arrays);

  {- -------------------------------------------
  (1) JvmtiGetLoadedClassesClosure::allocate() を呼び出して
      計算した個数分の配列を確保する.
      ---------------------------------------- -}

	    // Next, fill in the classes
	    closure.allocate();

  {- -------------------------------------------
  (1) 次に, 確保した配列に該当するクラスを格納していく
      (この処理は, 全てのクラスに対して JvmtiGetLoadedClassesClosure::add_with_loader() や
       JvmtiGetLoadedClassesClosure::add_for_basic_type_arrays() を呼び出すことで行う)
      ---------------------------------------- -}

	    SystemDictionary::classes_do(&JvmtiGetLoadedClassesClosure::add_with_loader);
	    Universe::basic_type_classes_do(&JvmtiGetLoadedClassesClosure::add_for_basic_type_arrays);

  {- -------------------------------------------
  (1) (ここまでが SystemDictionary_lock で排他された処理. 
       この後でロードされたものは結果には含まれないが, 
       少なくともある時点でのスナップショットを表しているということは保証される)
      ---------------------------------------- -}

	    // Drop the SystemDictionary_lock, so the results could be wrong from here,
	    // but we still have a snapshot.
	  }

  {- -------------------------------------------
  (1) JVMTI の Allocate() 関数を呼んで, 結果を返却するための配列を確保する.
      確保が失敗したらここでリターン.
      ---------------------------------------- -}

	  // Post results
	  jclass* result_list;
	  jvmtiError err = env->Allocate(closure.get_count() * sizeof(jclass),
	                                 (unsigned char**)&result_list);
	  if (err != JVMTI_ERROR_NONE) {
	    return err;
	  }

  {- -------------------------------------------
  (1) JvmtiGetLoadedClassesClosure::extract() を呼んで, 
      取得した結果を返却用の配列にコピーする.
      ---------------------------------------- -}

	  closure.extract(env, result_list);

  {- -------------------------------------------
  (1) その他の引数にも結果をセットしておく.
      ---------------------------------------- -}

	  *classCountPtr = closure.get_count();
	  *classesPtr = result_list;

  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	}
	
```


