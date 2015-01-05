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


```cpp
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

<div class="flow-abst"><pre>
* java.lang.Class のリフレクション機能

  java.lang.Class.getDeclaredClasses()
  -&gt; java.lang.Class.getDeclaredClasses0()
     -&gt; JVM_GetDeclaredClasses()
        -&gt; Reflection::check_for_inner_class()

  java.lang.Class.getSimpleName()
  -&gt; java.lang.Class.getComponentType()
     -&gt; JVM_GetComponentType()
        -&gt; Reflection::array_component_type()
  
  java.lang.Class.getCanonicalName()
  -&gt; java.lang.Class.getComponentType()
     -&gt; JVM_GetComponentType()
        -&gt; Reflection::array_component_type()

  java.lang.Class.getResourceAsStream()
  -&gt; java.lang.Class.resolveName()
     -&gt; java.lang.Class.getComponentType()
        -&gt; JVM_GetComponentType()
           -&gt; Reflection::array_component_type()

  java.lang.Class.getResource()
  -&gt; java.lang.Class.resolveName()
     -&gt; java.lang.Class.getComponentType()
        -&gt; JVM_GetComponentType()
           -&gt; Reflection::array_component_type()

  java.lang.Class.getDeclaringClass()
  -&gt; JVM_GetDeclaringClass()
     -&gt; instanceKlass::compute_enclosing_class()
        -&gt; instanceKlass::compute_enclosing_class_impl()
           -&gt; Reflection::check_for_inner_class()

  java.lang.Class.getDeclaredFields()
  -&gt; java.lang.Class.privateGetDeclaredFields()
     -&gt; java.lang.Class.getDeclaredFields0()
        -&gt; JVM_GetClassDeclaredFields()
           -&gt; Reflection::new_field()

  java.lang.Class.getDeclaredField()
  -&gt; java.lang.Class.privateGetDeclaredFields()
     -&gt; (同上)

  java.lang.Class.getFields()
  -&gt; java.lang.Class.privateGetPublicFields()
     -&gt; java.lang.Class.privateGetDeclaredFields()
        -&gt; (同上)
     -&gt; java.lang.Class.privateGetPublicFields() (再帰呼び出し)
        -&gt; (同上)

  java.lang.Class.getDeclaredMethods()
  -&gt; java.lang.Class.privateGetDeclaredMethods()
     -&gt; java.lang.Class.getDeclaredMethods0()
        -&gt; JVM_GetClassDeclaredMethods()
           -&gt; Reflection::new_method()

  java.lang.Class.getDeclaredMethod()
  -&gt; java.lang.Class.privateGetDeclaredMethods()
     -&gt; (同上)

  java.lang.Class.getMethods()
  -&gt; java.lang.Class.privateGetPublicMethods()
     -&gt; java.lang.Class.privateGetDeclaredMethods()
        -&gt; (同上)
     -&gt; java.lang.Class.privateGetPublicMethods() (再帰呼び出し)
        -&gt; (同上)

  java.lang.Class.getMethod()
  -&gt; java.lang.Class.getMethod0()
     -&gt; java.lang.Class.privateGetDeclaredMethods()
        -&gt; (同上)
     -&gt; java.lang.Class.getMethod0() (再帰呼び出し)
        -&gt; (同上)

  java.lang.Class.getConstructors()
  -&gt; java.lang.Class.privateGetDeclaredConstructors()
     -&gt; java.lang.Class.getDeclaredConstructors0()
        -&gt; JVM_GetClassDeclaredConstructors()
           -&gt; Reflection::new_constructor()

  java.lang.Class.getConstructors()
  -&gt; java.lang.Class.privateGetDeclaredConstructors()
     -&gt; (同上)

  java.lang.Class.getDeclaredConstructors()
  -&gt; java.lang.Class.privateGetDeclaredConstructors()
     -&gt; (同上)

  java.lang.Class.newInstance()
  -&gt; java.lang.Class.newInstance0()
     -&gt; java.lang.Class.getConstructor0()
        -&gt; java.lang.Class.privateGetDeclaredConstructors()
           -&gt; (同上)

  java.lang.Class.getConstructor()
  -&gt; java.lang.Class.getConstructor0()
     -&gt; (同上)

  java.lang.Class.getDeclaredConstructor()
  -&gt; java.lang.Class.getConstructor0()
     -&gt; (同上)

* java.lang.reflect.Array の処理

  java.lang.reflect.Array.get()
  -&gt; Java_java_lang_reflect_Array_get()
     -&gt; JVM_GetArrayElement()
        -&gt; Reflection::array_get()
        -&gt; Reflection::box()

  java.lang.reflect.Array.getBoolean()
  -&gt; Java_java_lang_reflect_Array_getBoolean()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; Reflection::array_get()
        -&gt; Reflection::widen()

  java.lang.reflect.Array.getByte()
  -&gt; Java_java_lang_reflect_Array_getByte()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getChar()
  -&gt; Java_java_lang_reflect_Array_getChar()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getShort()
  -&gt; Java_java_lang_reflect_Array_getShort()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getInt()
  -&gt; Java_java_lang_reflect_Array_getInt()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getLong()
  -&gt; Java_java_lang_reflect_Array_getLong()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getFloat()
  -&gt; Java_java_lang_reflect_Array_getFloat()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.getDouble()
  -&gt; Java_java_lang_reflect_Array_getDouble()
     -&gt; JVM_GetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.set()
  -&gt; Java_java_lang_reflect_Array_set()
     -&gt; JVM_SetArrayElement()
        -&gt; Reflection::unbox_for_regular_object()
        -&gt; Reflection::unbox_for_primitive()
        -&gt; Reflection::array_set()

  java.lang.reflect.Array.setBoolean()
  -&gt; Java_java_lang_reflect_Array_setBoolean()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; Reflection::array_set()
  
  java.lang.reflect.Array.setByte()
  -&gt; Java_java_lang_reflect_Array_setByte()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setChar()
  -&gt; Java_java_lang_reflect_Array_setChar()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setShort()
  -&gt; Java_java_lang_reflect_Array_setShort()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setInt()
  -&gt; Java_java_lang_reflect_Array_setInt()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setLong()
  -&gt; Java_java_lang_reflect_Array_setLong()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setFloat()
  -&gt; Java_java_lang_reflect_Array_setFloat()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.setDouble()
  -&gt; Java_java_lang_reflect_Array_setDouble()
     -&gt; JVM_SetPrimitiveArrayElement()
        -&gt; (同上)

  java.lang.reflect.Array.newArray()
  -&gt; Java_java_lang_reflect_Array_newArray()
     -&gt; JVM_NewArray()
        -&gt; Reflection::reflect_new_array()

  java.lang.reflect.Array.multiNewArray()
  -&gt; Java_java_lang_reflect_Array_multiNewArray()
     -&gt; JVM_NewMultiArray()
        -&gt; Reflection::reflect_new_multi_array()

* java.lang.reflect.Method の処理

  java.lang.reflect.Method.invoke()
  -&gt; sun.reflect.DelegatingMethodAccessorImpl.invoke()
     -&gt; sun.reflect.NativeMethodAccessorImpl.invoke()
        -&gt; sun.reflect.NativeMethodAccessorImpl.invoke0()
           -&gt; Java_sun_reflect_NativeMethodAccessorImpl_invoke0()
              -&gt; JVM_InvokeMethod()
                 -&gt; Reflection::invoke_method()

* java.lang.reflect.Constructor の処理

  java.lang.reflect.Constructor.newInstance()
  -&gt; sun.reflect.DelegatingConstructorAccessorImpl.newInstance()
     -&gt; sun.reflect.NativeConstructorAccessorImpl.newInstance()
        -&gt; sun.reflect.NativeConstructorAccessorImpl.newInstance0()
           -&gt; Java_sun_reflect_NativeConstructorAccessorImpl_newInstance0()
              -&gt; JVM_NewInstanceFromConstructor()
                 -&gt; Reflection::invoke_constructor()

* JNI によるReflection処理

  ToReflectedMethod()
  -&gt; jni_ToReflectedMethod()
     -&gt; Reflection::new_constructor()
     -&gt; Reflection::new_method()

  ToReflectedField()
  -&gt; jni_ToReflectedField()
     -&gt; Reflection::new_field()

* java.lang.invoke.MethodHandle 関係の処理

  java.lang.invoke.MethodHandleNatives.getTarget()
  -&gt; MHN_getTarget()
     -&gt; MethodHandles::encode_target()
        -&gt; Reflection::new_constructor()
        -&gt; Reflection::new_method()

  java.lang.invoke.MethodHandleNatives.init(DirectMethodHandle self, Object ref, boolean doDispatch, Class&lt;?&gt; caller)
  -&gt; MHN_init_DMH()
     -&gt; Reflection::verify_field_access()
     -&gt; Reflection::is_same_package_member()
        -&gt; instanceKlass::is_same_package_member_impl()
           -&gt; instanceKlass::compute_enclosing_class_impl()
              -&gt; Reflection::check_for_inner_class()

  java.lang.invoke.MethodHandleNatives.resolve()
  -&gt; MHN_resolve_Mem()
     -&gt; Reflection::verify_class_access()

* クラスやフィールドに対するアクセス権のチェック処理

  ciEnv::check_klass_accessibility()
  -&gt; Reflection::verify_class_access()

  ClassFileParser::check_super_class_access()
  -&gt; Reflection::verify_class_access()
  
  ClassFileParser::check_super_interface_access()
  -&gt; Reflection::verify_class_access()
  
  ClassFileParser::check_final_method_override()
  -&gt; Reflection::verify_field_access()
  
  LinkResolver::check_klass_accessability()
  -&gt; Reflection::verify_class_access()
  
  LinkResolver::check_method_accessability()
  -&gt; Reflection::verify_field_access()
  
  LinkResolver::check_field_accessability()
  -&gt; Reflection::verify_field_access()
  
* sun.reflect.ConstantPool の処理

  (?? 使用箇所が見当たらない)
  -&gt; sun.reflect.ConstantPool.getMethodAt()
     -&gt; sun.reflect.ConstantPool.getMethodAt0()
        -&gt; Java_sun_reflect_ConstantPool_getMethodAt0()
           -&gt; JVM_ConstantPoolGetMethodAt()
              -&gt; get_method_at_helper()
                 -&gt; Reflection::new_method()
                 -&gt; Reflection::new_constructor()

  (?? 使用箇所が見当たらない)
  -&gt; sun.reflect.ConstantPool.getMethodAtIfLoaded()
     -&gt; sun.reflect.ConstantPool.getMethodAtIfLoaded0()
        -&gt; Java_sun_reflect_ConstantPool_getMethodAtIfLoaded0()
           -&gt; JVM_ConstantPoolGetMethodAtIfLoaded()
              -&gt; get_method_at_helper()
                 -&gt; Reflection::new_method()
                 -&gt; Reflection::new_constructor()

  (?? 使用箇所が見当たらない)
  -&gt; sun.reflect.ConstantPool.getFieldAt()
     -&gt; sun.reflect.ConstantPool.getFieldAt0()
        -&gt; Java_sun_reflect_ConstantPool_getFieldAt0()
           -&gt; JVM_ConstantPoolGetFieldAt()
              -&gt; get_field_at_helper()
                 -&gt; Reflection::new_field()
     
  (?? 使用箇所が見当たらない)
  -&gt; sun.reflect.ConstantPool.getFieldAtIfLoaded()
     -&gt; sun.reflect.ConstantPool.getFieldAtIfLoaded0()
        -&gt; Java_sun_reflect_ConstantPool_getFieldAtIfLoaded0()
           -&gt; JVM_ConstantPoolGetFieldAtIfLoaded()
              -&gt; get_field_at_helper()
                 -&gt; Reflection::new_field()

* 1.3 以前の JVM との互換性を取るための機能 (これらは #ifdef SUPPORT_OLD_REFLECTION 時にのみ定義される機能)

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassField()
     -&gt; Reflection::reflect_field()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassFields()
     -&gt; Reflection::reflect_fields()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassMethod()
     -&gt; Reflection::reflect_method()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassMethods()
     -&gt; Reflection::reflect_methods()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_NewInstance()
     -&gt; Reflection::verify_class_access()
     -&gt; Reflection::verify_field_access()
     
  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassConstructors()
     -&gt; Reflection::reflect_constructors()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetClassConstructor()
     -&gt; Reflection::reflect_constructor()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetField()
     -&gt; Reflection::resolve_field()
     -&gt; Reflection::field_get()
     -&gt; Reflection::box()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_GetPrimitiveField()
     -&gt; Reflection::resolve_field()
     -&gt; Reflection::field_get()
     -&gt; Reflection::widen()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_SetField()
     -&gt; Reflection::resolve_field()
     -&gt; Reflection::unbox_for_regular_object()
     -&gt; Reflection::field_set()
     -&gt; Reflection::unbox_for_primitive()
     -&gt; Reflection::field_set()

  (?? 使用箇所が見当たらない)
  -&gt; JVM_SetPrimitiveField()
     -&gt; Reflection::resolve_field()
     -&gt; Reflection::field_set()

* バイトコードの verification 処理 (See: <a href="no7882amm.html">here</a> for details)

  VerifyClassCodes()
  -&gt; VerifyClass()
     -&gt; VerifyClassForMajorVersion()
        -&gt; verify_method()
           -&gt; verify_opcode_operands()
              -&gt; set_protected()
                 -&gt; JVM_IsSameClassPackage()
                    -&gt; Reflection::is_same_class_package()

  VerifyClassCodesForMajorVersion()
  -&gt; VerifyClassForMajorVersion()
     -&gt; (同上)
</pre></div>

### 内部構造(Internal structure)
定義されている public メソッドは, 以下の通り.

なお, 直接的にリフレクション機能を実現するためのメソッドだけではなく, 
クラスやフィールドに対するアクセス権をチェックするメソッドも備えている.

  * Reflection::verify_class_access()
  * Reflection::verify_field_access()

何故 Reflection クラスがアクセス権に関係するかというと, 
リフレクション機能で生成した FieldAccessorImpl オブジェクトや MethodAccessorImpl オブジェクトについては
(通常のアクセス規則を無視して) フィールドやメソッドにアクセスできるようにするため.


```cpp
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


```cpp
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


```cpp
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
