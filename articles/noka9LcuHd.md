---
layout: default
title: JvmtiGetLoadedClasses クラス (JvmtiGetLoadedClasses, 及びその補助クラス(JvmtiGetLoadedClassesClosure))
---
[Top](../index.html)

#### JvmtiGetLoadedClasses クラス (JvmtiGetLoadedClasses, 及びその補助クラス(JvmtiGetLoadedClassesClosure))

これらは, JVMTI の関数を実装するために使われているクラス.
より具体的に言うと, GetLoadedClasses() 関数と GetClassLoaderClasses() 関数を実装するためのクラス (See: [here](no29354IF.html) for details).


### クラス一覧(class list)

  * [JvmtiGetLoadedClasses](#noqs4uv4LH)
  * [JvmtiGetLoadedClassesClosure](#no9XZtn2mY)


---
## <a name="noqs4uv4LH" id="noqs4uv4LH">JvmtiGetLoadedClasses</a>

### 概要(Summary)
JVMTI の関数 (より具体的に言うと, GetLoadedClasses() 関数と GetClassLoaderClasses() 関数) 
のための関数を納めた名前空間(AllStatic クラス) (See: [here](no29354IF.html) for details).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.hpp))
    class JvmtiGetLoadedClasses : AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* JvmtiEnv::GetLoadedClasses()
* JvmtiEnv::GetClassLoaderClasses()

(というか JvmtiEnv は JvmtiGetLoadedClasses に処理を丸投げしているだけ...)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEnv.cpp))
    // class_count_ptr - pre-checked for NULL
    // classes_ptr - pre-checked for NULL
    jvmtiError
    JvmtiEnv::GetLoadedClasses(jint* class_count_ptr, jclass** classes_ptr) {
      return JvmtiGetLoadedClasses::getLoadedClasses(this, class_count_ptr, classes_ptr);
    } /* end GetLoadedClasses */
    
    
    // initiating_loader - NULL is a valid value, must be checked
    // class_count_ptr - pre-checked for NULL
    // classes_ptr - pre-checked for NULL
    jvmtiError
    JvmtiEnv::GetClassLoaderClasses(jobject initiating_loader, jint* class_count_ptr, jclass** classes_ptr) {
      return JvmtiGetLoadedClasses::getClassLoaderClasses(this, initiating_loader,
                                                      class_count_ptr, classes_ptr);
    } /* end GetClassLoaderClasses */
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.


```
    ((cite: hotspot/src/share/vm/prims/jvmtiGetLoadedClasses.hpp))
      static jvmtiError getLoadedClasses(JvmtiEnv *env, jint* classCountPtr, jclass** classesPtr);
      static jvmtiError getClassLoaderClasses(JvmtiEnv *env, jobject initiatingLoader,
                                              jint* classCountPtr, jclass** classesPtr);
```




### 詳細(Details)
See: [here](../doxygen/classJvmtiGetLoadedClasses.html) for details

---
## <a name="no9XZtn2mY" id="no9XZtn2mY">JvmtiGetLoadedClassesClosure</a>

### 概要(Summary)
JvmtiGetLoadedClasses クラス用の補助クラス(StackObjクラス).

指定されたクラスローダがロードしたクラス(あるいはクラスローダに関係なくロードされた全てのクラス)の一覧を取得する
(合わせて該当するクラス数も取得する).

### 使われ方(Usage)
JvmtiGetLoadedClasses::getLoadedClasses() 内, 及び
JvmtiGetLoadedClasses::getClassLoaderClasses() 内で(のみ)使用されている (See: [here](no29354IF.html) for details).

なお, この処理で使用する SystemDictionary::classes_do() や Universe::basic_type_classes_do() は
Closure ではなく関数ポインタを受け取るため, JvmtiGetLoadedClassesClosure は直接は使用できない.
そのため, JavaThread オブジェクト内に JvmtiGetLoadedClasses 型の
_jvmti_get_loaded_classes_closure というフィールドが用意されている
(処理の開始前に カレントスレッドのこのフィールドに使用する JvmtiGetLoadedClasses オブジェクトが登録され,
SystemDictionary::classes_do() や Universe::basic_type_classes_do() に実際に引き渡される関数は,
カレントスレッドのこのフィールドから JvmtiGetLoadedClasses オブジェクトを取得して処理を行っている.).




### 詳細(Details)
See: [here](../doxygen/classJvmtiGetLoadedClassesClosure.html) for details

---
