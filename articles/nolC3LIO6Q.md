---
layout: default
title: Reflection クラス 
---
[Top](../index.html)

#### Reflection クラス 



---
## <a name="noAZUbjPgC" id="noAZUbjPgC">Reflection</a>

### 概要(Summary)
Java のリフレクション(Reflection)機能を実現するためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)) (See: [here](no1904Wlh.html) and [here](nodj-PTmlM.html) for details).

(なおコメントによると, 
「JDK 1.4 からは dynamic bytecode generation 機能が付いたため, 
ほとんどの Reflection 機能が Java コードで実装されるようになり, このクラスの役割は大幅に縮小した.
現在でも Array クラスだけはバイトコードで書き換えできないが, 
それもできるようになれば Reflection クラスのほとんどの部分は不要になる」, とのこと.)

(<= このクラスは現状では Reflection 用の Java オブジェクト (java.lang.reflect オブジェクト) を生成するファクトリクラスとしての役割と, 
生成した java.lang.reflect オブジェクトから呼び出される補助クラスとしての役割がある.
コメントの意味は, ファクトリクラスとしての役割だけが残って, 
生成した Reflection 用の Java オブジェクトがこのクラスを呼び出す必要性はなくなる, 
ということだと思われる)


```
    ((cite: hotspot/src/share/vm/runtime/reflection.hpp))
    // Class Reflection contains utility methods needed for implementing the
    // reflection api.
    //
    // Used by functions in the JVM interface.
    //
    // NOTE that in JDK 1.4 most of reflection is now implemented in Java
    // using dynamic bytecode generation. The Array class has not yet been
    // rewritten using bytecodes; if it were, most of the rest of this
    // class could go away, as well as a few more entry points in jvm.cpp.
    
    class FieldStream;
    
    class Reflection: public AllStatic {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

```
* java.lang.Class のリフレクション機能

  java.lang.Class.getDeclaredClasses()
  -> java.lang.Class.getDeclaredClasses0()
     -> JVM_GetDeclaredClasses()
        -> Reflection::check_for_inner_class()

  java.lang.Class.getSimpleName()
  -> java.lang.Class.getComponentType()
     -> JVM_GetComponentType()
        -> Reflection::array_component_type()
  
  java.lang.Class.getCanonicalName()
  -> java.lang.Class.getComponentType()
     -> JVM_GetComponentType()
        -> Reflection::array_component_type()

  java.lang.Class.getResourceAsStream()
  -> java.lang.Class.resolveName()
     -> java.lang.Class.getComponentType()
        -> JVM_GetComponentType()
           -> Reflection::array_component_type()

  java.lang.Class.getResource()
  -> java.lang.Class.resolveName()
     -> java.lang.Class.getComponentType()
        -> JVM_GetComponentType()
           -> Reflection::array_component_type()

  java.lang.Class.getDeclaringClass()
  -> JVM_GetDeclaringClass()
     -> instanceKlass::compute_enclosing_class()
        -> instanceKlass::compute_enclosing_class_impl()
           -> Reflection::check_for_inner_class()

  java.lang.Class.getDeclaredFields()
  -> java.lang.Class.privateGetDeclaredFields()
     -> java.lang.Class.getDeclaredFields0()
        -> JVM_GetClassDeclaredFields()
           -> Reflection::new_field()

  java.lang.Class.getDeclaredField()
  -> java.lang.Class.privateGetDeclaredFields()
     -> (同上)

  java.lang.Class.getFields()
  -> java.lang.Class.privateGetPublicFields()
     -> java.lang.Class.privateGetDeclaredFields()
        -> (同上)
     -> java.lang.Class.privateGetPublicFields() (再帰呼び出し)
        -> (同上)

  java.lang.Class.getDeclaredMethods()
  -> java.lang.Class.privateGetDeclaredMethods()
     -> java.lang.Class.getDeclaredMethods0()
        -> JVM_GetClassDeclaredMethods()
           -> Reflection::new_method()

  java.lang.Class.getDeclaredMethod()
  -> java.lang.Class.privateGetDeclaredMethods()
     -> (同上)

  java.lang.Class.getMethods()
  -> java.lang.Class.privateGetPublicMethods()
     -> java.lang.Class.privateGetDeclaredMethods()
        -> (同上)
     -> java.lang.Class.privateGetPublicMethods() (再帰呼び出し)
        -> (同上)

  java.lang.Class.getMethod()
  -> java.lang.Class.getMethod0()
     -> java.lang.Class.privateGetDeclaredMethods()
        -> (同上)
     -> java.lang.Class.getMethod0() (再帰呼び出し)
        -> (同上)

  java.lang.Class.getConstructors()
  -> java.lang.Class.privateGetDeclaredConstructors()
     -> java.lang.Class.getDeclaredConstructors0()
        -> JVM_GetClassDeclaredConstructors()
           -> Reflection::new_constructor()

  java.lang.Class.getConstructors()
  -> java.lang.Class.privateGetDeclaredConstructors()
     -> (同上)

  java.lang.Class.getDeclaredConstructors()
  -> java.lang.Class.privateGetDeclaredConstructors()
     -> (同上)

  java.lang.Class.newInstance()
  -> java.lang.Class.newInstance0()
     -> java.lang.Class.getConstructor0()
        -> java.lang.Class.privateGetDeclaredConstructors()
           -> (同上)

  java.lang.Class.getConstructor()
  -> java.lang.Class.getConstructor0()
     -> (同上)

  java.lang.Class.getDeclaredConstructor()
  -> java.lang.Class.getConstructor0()
     -> (同上)

* java.lang.reflect.Array の処理

  java.lang.reflect.Array.get()
  -> Java_java_lang_reflect_Array_get()
     -> JVM_GetArrayElement()
        -> Reflection::array_get()
        -> Reflection::box()

  java.lang.reflect.Array.getBoolean()
  -> Java_java_lang_reflect_Array_getBoolean()
     -> JVM_GetPrimitiveArrayElement()
        -> Reflection::array_get()
        -> Reflection::widen()

  java.lang.reflect.Array.getByte()
  -> Java_java_lang_reflect_Array_getByte()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getChar()
  -> Java_java_lang_reflect_Array_getChar()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getShort()
  -> Java_java_lang_reflect_Array_getShort()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getInt()
  -> Java_java_lang_reflect_Array_getInt()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getLong()
  -> Java_java_lang_reflect_Array_getLong()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getFloat()
  -> Java_java_lang_reflect_Array_getFloat()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.getDouble()
  -> Java_java_lang_reflect_Array_getDouble()
     -> JVM_GetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.set()
  -> Java_java_lang_reflect_Array_set()
     -> JVM_SetArrayElement()
        -> Reflection::unbox_for_regular_object()
        -> Reflection::unbox_for_primitive()
        -> Reflection::array_set()

  java.lang.reflect.Array.setBoolean()
  -> Java_java_lang_reflect_Array_setBoolean()
     -> JVM_SetPrimitiveArrayElement()
        -> Reflection::array_set()
  
  java.lang.reflect.Array.setByte()
  -> Java_java_lang_reflect_Array_setByte()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setChar()
  -> Java_java_lang_reflect_Array_setChar()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setShort()
  -> Java_java_lang_reflect_Array_setShort()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setInt()
  -> Java_java_lang_reflect_Array_setInt()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setLong()
  -> Java_java_lang_reflect_Array_setLong()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setFloat()
  -> Java_java_lang_reflect_Array_setFloat()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.setDouble()
  -> Java_java_lang_reflect_Array_setDouble()
     -> JVM_SetPrimitiveArrayElement()
        -> (同上)

  java.lang.reflect.Array.newArray()
  -> Java_java_lang_reflect_Array_newArray()
     -> JVM_NewArray()
        -> Reflection::reflect_new_array()

  java.lang.reflect.Array.multiNewArray()
  -> Java_java_lang_reflect_Array_multiNewArray()
     -> JVM_NewMultiArray()
        -> Reflection::reflect_new_multi_array()

* java.lang.reflect.Method の処理

  java.lang.reflect.Method.invoke()
  -> sun.reflect.DelegatingMethodAccessorImpl.invoke()
     -> sun.reflect.NativeMethodAccessorImpl.invoke()
        -> sun.reflect.NativeMethodAccessorImpl.invoke0()
           -> Java_sun_reflect_NativeMethodAccessorImpl_invoke0()
              -> JVM_InvokeMethod()
                 -> Reflection::invoke_method()

* java.lang.reflect.Constructor の処理

  java.lang.reflect.Constructor.newInstance()
  -> sun.reflect.DelegatingConstructorAccessorImpl.newInstance()
     -> sun.reflect.NativeConstructorAccessorImpl.newInstance()
        -> sun.reflect.NativeConstructorAccessorImpl.newInstance0()
           -> Java_sun_reflect_NativeConstructorAccessorImpl_newInstance0()
              -> JVM_NewInstanceFromConstructor()
                 -> Reflection::invoke_constructor()

* JNI によるReflection処理

  ToReflectedMethod()
  -> jni_ToReflectedMethod()
     -> Reflection::new_constructor()
     -> Reflection::new_method()

  ToReflectedField()
  -> jni_ToReflectedField()
     -> Reflection::new_field()

* java.lang.invoke.MethodHandle 関係の処理

  java.lang.invoke.MethodHandleNatives.getTarget()
  -> MHN_getTarget()
     -> MethodHandles::encode_target()
        -> Reflection::new_constructor()
        -> Reflection::new_method()

  java.lang.invoke.MethodHandleNatives.init(DirectMethodHandle self, Object ref, boolean doDispatch, Class<?> caller)
  -> MHN_init_DMH()
     -> Reflection::verify_field_access()
     -> Reflection::is_same_package_member()
        -> instanceKlass::is_same_package_member_impl()
           -> instanceKlass::compute_enclosing_class_impl()
              -> Reflection::check_for_inner_class()

  java.lang.invoke.MethodHandleNatives.resolve()
  -> MHN_resolve_Mem()
     -> Reflection::verify_class_access()

* クラスやフィールドに対するアクセス権のチェック処理

  ciEnv::check_klass_accessibility()
  -> Reflection::verify_class_access()

  ClassFileParser::check_super_class_access()
  -> Reflection::verify_class_access()
  
  ClassFileParser::check_super_interface_access()
  -> Reflection::verify_class_access()
  
  ClassFileParser::check_final_method_override()
  -> Reflection::verify_field_access()
  
  LinkResolver::check_klass_accessability()
  -> Reflection::verify_class_access()
  
  LinkResolver::check_method_accessability()
  -> Reflection::verify_field_access()
  
  LinkResolver::check_field_accessability()
  -> Reflection::verify_field_access()
  
* sun.reflect.ConstantPool の処理

  (?? 使用箇所が見当たらない)
  -> sun.reflect.ConstantPool.getMethodAt()
     -> sun.reflect.ConstantPool.getMethodAt0()
        -> Java_sun_reflect_ConstantPool_getMethodAt0()
           -> JVM_ConstantPoolGetMethodAt()
              -> get_method_at_helper()
                 -> Reflection::new_method()
                 -> Reflection::new_constructor()

  (?? 使用箇所が見当たらない)
  -> sun.reflect.ConstantPool.getMethodAtIfLoaded()
     -> sun.reflect.ConstantPool.getMethodAtIfLoaded0()
        -> Java_sun_reflect_ConstantPool_getMethodAtIfLoaded0()
           -> JVM_ConstantPoolGetMethodAtIfLoaded()
              -> get_method_at_helper()
                 -> Reflection::new_method()
                 -> Reflection::new_constructor()

  (?? 使用箇所が見当たらない)
  -> sun.reflect.ConstantPool.getFieldAt()
     -> sun.reflect.ConstantPool.getFieldAt0()
        -> Java_sun_reflect_ConstantPool_getFieldAt0()
           -> JVM_ConstantPoolGetFieldAt()
              -> get_field_at_helper()
                 -> Reflection::new_field()
     
  (?? 使用箇所が見当たらない)
  -> sun.reflect.ConstantPool.getFieldAtIfLoaded()
     -> sun.reflect.ConstantPool.getFieldAtIfLoaded0()
        -> Java_sun_reflect_ConstantPool_getFieldAtIfLoaded0()
           -> JVM_ConstantPoolGetFieldAtIfLoaded()
              -> get_field_at_helper()
                 -> Reflection::new_field()

* 1.3 以前の JVM との互換性を取るための機能 (これらは #ifdef SUPPORT_OLD_REFLECTION 時にのみ定義される機能)

  (?? 使用箇所が見当たらない)
  -> JVM_GetClassField()
     -> Reflection::reflect_field()

  (?? 使用箇所が見当たらない)
  -> JVM_GetClassFields()
     -> Reflection::reflect_fields()

  (?? 使用箇所が見当たらない)
  -> JVM_GetClassMethod()
     -> Reflection::reflect_method()

  (?? 使用箇所が見当たらない)
  -> JVM_GetClassMethods()
     -> Reflection::reflect_methods()

  (?? 使用箇所が見当たらない)
  -> JVM_NewInstance()
     -> Reflection::verify_class_access()
     -> Reflection::verify_field_access()
     
  (?? 使用箇所が見当たらない)
  -> JVM_GetClassConstructors()
     -> Reflection::reflect_constructors()

  (?? 使用箇所が見当たらない)
  -> JVM_GetClassConstructor()
     -> Reflection::reflect_constructor()

  (?? 使用箇所が見当たらない)
  -> JVM_GetField()
     -> Reflection::resolve_field()
     -> Reflection::field_get()
     -> Reflection::box()

  (?? 使用箇所が見当たらない)
  -> JVM_GetPrimitiveField()
     -> Reflection::resolve_field()
     -> Reflection::field_get()
     -> Reflection::widen()

  (?? 使用箇所が見当たらない)
  -> JVM_SetField()
     -> Reflection::resolve_field()
     -> Reflection::unbox_for_regular_object()
     -> Reflection::field_set()
     -> Reflection::unbox_for_primitive()
     -> Reflection::field_set()

  (?? 使用箇所が見当たらない)
  -> JVM_SetPrimitiveField()
     -> Reflection::resolve_field()
     -> Reflection::field_set()

* 

  (?? 使用箇所が見当たらない)
  -> VerifyClassCodes()
     -> VerifyClass()
        -> VerifyClassForMajorVersion()
           -> verify_method()
              -> verify_opcode_operands()
                 -> set_protected()
                    -> JVM_IsSameClassPackage()
                       -> Reflection::is_same_class_package()

  (?? 使用箇所が見当たらない)
  -> VerifyClassCodesForMajorVersion()
     -> VerifyClassForMajorVersion()
        -> (同上)
```

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の通り.

なお, 直接的にリフレクション機能を実現するためのメソッドだけではなく, 
クラスやフィールドに対するアクセス権をチェックするメソッドも備えている.

  * Reflection::verify_class_access()
  * Reflection::verify_field_access()

何故 Reflection クラスがアクセス権に関係するかというと, 
リフレクション機能で生成した FieldAccessorImpl オブジェクトや MethodAccessorImpl オブジェクトについては
(通常のアクセス規則を無視して) フィールドやメソッドにアクセスできるようにするため.


```
    ((cite: hotspot/src/share/vm/runtime/reflection.hpp))
      // Boxing. Returns boxed value of appropriate type. Throws IllegalArgumentException.
      static oop box(jvalue* v, BasicType type, TRAPS);
      // Unboxing. Returns type code and sets value.
      static BasicType unbox_for_primitive(oop boxed_value, jvalue* value, TRAPS);
      static BasicType unbox_for_regular_object(oop boxed_value, jvalue* value);
    
      // Widening of basic types. Throws IllegalArgumentException.
      static void widen(jvalue* value, BasicType current_type, BasicType wide_type, TRAPS);
    
      // Reflective array access. Returns type code. Throws ArrayIndexOutOfBoundsException.
      static BasicType array_get(jvalue* value, arrayOop a, int index, TRAPS);
      static void      array_set(jvalue* value, arrayOop a, int index, BasicType value_type, TRAPS);
      // Returns mirror on array element type (NULL for basic type arrays and non-arrays).
      static oop       array_component_type(oop mirror, TRAPS);
    
      // Object creation
      static arrayOop reflect_new_array(oop element_mirror, jint length, TRAPS);
      static arrayOop reflect_new_multi_array(oop element_mirror, typeArrayOop dimensions, TRAPS);
    
      // Verification
      static bool     verify_class_access(klassOop current_class, klassOop new_class, bool classloader_only);
    
      static bool     verify_field_access(klassOop current_class,
                                          klassOop resolved_class,
                                          klassOop field_class,
                                          AccessFlags access,
                                          bool classloader_only,
                                          bool protected_restriction = false);
      static bool     is_same_class_package(klassOop class1, klassOop class2);
      static bool     is_same_package_member(klassOop class1, klassOop class2, TRAPS);
    
      static bool can_relax_access_check_for(
        klassOop accessor, klassOop accesee, bool classloader_only);
    
      // inner class reflection
      // raise an ICCE unless the required relationship can be proven to hold
      // If inner_is_member, require the inner to be a member of the outer.
      // If !inner_is_member, require the inner to be anonymous (a non-member).
      // Caller is responsible for figuring out in advance which case must be true.
      static void check_for_inner_class(instanceKlassHandle outer, instanceKlassHandle inner,
                                        bool inner_is_member, TRAPS);
    
      //
      // Support for reflection based on dynamic bytecode generation (JDK 1.4)
      //
    
      // Create a java.lang.reflect.Method object based on a method
      static oop new_method(methodHandle method, bool intern_name, bool for_constant_pool_access, TRAPS);
      // Create a java.lang.reflect.Constructor object based on a method
      static oop new_constructor(methodHandle method, TRAPS);
      // Create a java.lang.reflect.Field object based on a field descriptor
      static oop new_field(fieldDescriptor* fd, bool intern_name, TRAPS);
```


```
    ((cite: hotspot/src/share/vm/runtime/reflection.hpp))
      //---------------------------------------------------------------------------
      //
      // Support for old native code-based reflection (pre-JDK 1.4)
      //
      // NOTE: the method and constructor invocation code is still used
      // for startup time reasons; see reflectionCompat.hpp.
      //
      //---------------------------------------------------------------------------
    
    #ifdef SUPPORT_OLD_REFLECTION
    ...
    public:
      // Field lookup and verification.
      static bool      resolve_field(Handle field_mirror, Handle& receiver, fieldDescriptor* fd, bool check_final, TRAPS);
    
      // Reflective field access. Returns type code. Throws IllegalArgumentException.
      static BasicType field_get(jvalue* value, fieldDescriptor* fd, Handle receiver);
      static void      field_set(jvalue* value, fieldDescriptor* fd, Handle receiver, BasicType value_type, TRAPS);
    
      // Reflective lookup of fields. Returns java.lang.reflect.Field instances.
      static oop         reflect_field(oop mirror, Symbol* field_name, jint which, TRAPS);
      static objArrayOop reflect_fields(oop mirror, jint which, TRAPS);
    
      // Reflective lookup of methods. Returns java.lang.reflect.Method instances.
      static oop         reflect_method(oop mirror, Symbol* method_name, objArrayHandle types, jint which, TRAPS);
      static objArrayOop reflect_methods(oop mirror, jint which, TRAPS);
    
      // Reflective lookup of constructors. Returns java.lang.reflect.Constructor instances.
      static oop         reflect_constructor(oop mirror, objArrayHandle types, jint which, TRAPS);
      static objArrayOop reflect_constructors(oop mirror, jint which, TRAPS);
    
      // Method invokation through java.lang.reflect.Method
      static oop      invoke_method(oop method_mirror, Handle receiver, objArrayHandle args, TRAPS);
      // Method invokation through java.lang.reflect.Constructor
      static oop      invoke_constructor(oop method_mirror, objArrayHandle args, TRAPS);
    #endif /* SUPPORT_OLD_REFLECTION */
```

### 備考(Notes)
JDK 1.4 より前の時代のコードは, 現在は #ifdef SUPPORT_OLD_REFLECTION で囲われており, いつでも消せる体制になっている.
ただし, 現在は SUPPORT_OLD_REFLECTION は #define されている.

コメントによると, この理由は以下の通り.

「JDK 1.4 で dynamic bytecode generation が可能になったことで, 
Reflection クラスのネイティブコードは削除されることが期待されたが, 
スタブクラスのロードにより起動時間が長期化するという問題が生じた.
このため, JVM_InvokeMethod や JVM_NewInstanceFromConstructor 等については依然として残されている.

将来的にこの問題が解決されれば, product 版からは #define SUPPORT_OLD_REFLECTION が消去され, 
JVM_InvokeMethod や JVM_NewInstanceFromConstructor は定義されなくなる予定.

(なお, product 版以外では過去の JDK を用いたベンチマーク用途等のために残される予定)」


```
    ((cite: hotspot/src/share/vm/runtime/reflectionCompat.hpp))
    // During the development of the JDK 1.4 reflection implementation
    // based on dynamic bytecode generation, it was hoped that the bulk of
    // the native code for reflection could be removed. Unfortunately
    // there is currently a significant cost associated with loading the
    // stub classes which impacts startup time. Until this cost can be
    // reduced, the JVM entry points JVM_InvokeMethod and
    // JVM_NewInstanceFromConstructor are still needed; these and their
    // dependents currently constitute the bulk of the native code for
    // reflection. If this cost is reduced in the future, the
    // NativeMethodAccessorImpl and NativeConstructorAccessorImpl classes
    // can be removed from sun.reflect and all of the code guarded by this
    // flag removed from the product build. (Non-product builds,
    // specifically the "optimized" target, would retain the code so they
    // could be dropped into earlier JDKs for comparative benchmarking.)
    
    //#ifndef PRODUCT
    # define SUPPORT_OLD_REFLECTION
    //#endif
```




### 詳細(Details)
See: [here](../doxygen/classReflection.html) for details

---
