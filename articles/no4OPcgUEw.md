---
layout: default
title: constantTag クラス 
---
[Top](../index.html)

#### constantTag クラス 



---
## <a name="no1ZvUzT2e" id="no1ZvUzT2e">constantTag</a>

### 概要(Summary)
constant pool 中のタグ情報を表すためのユーティリティ・クラス.
1つの constantTag オブジェクトが 1つのタグ情報に対応する.

(なおここでの タグ(tag) とは, constant pool 内で要素の型を表す u1 値のこと.
 (e.g. CONSTANT_Class, CONSTANT_NameAndType, etc))


```cpp
    ((cite: hotspot/src/share/vm/utilities/constantTag.hpp))
    // constant tags in Java .class files
```


```cpp
    ((cite: hotspot/src/share/vm/utilities/constantTag.hpp))
    class constantTag VALUE_OBJ_CLASS_SPEC {
```

### 内部構造(Internal structure)
タグのうち, JVM 標準のタグについては jvm.h で定義されている.


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.h))
        JVM_CONSTANT_Utf8 = 1,
        JVM_CONSTANT_Unicode,               /* unused */
        JVM_CONSTANT_Integer,
        JVM_CONSTANT_Float,
        JVM_CONSTANT_Long,
        JVM_CONSTANT_Double,
        JVM_CONSTANT_Class,
        JVM_CONSTANT_String,
        JVM_CONSTANT_Fieldref,
        JVM_CONSTANT_Methodref,
        JVM_CONSTANT_InterfaceMethodref,
        JVM_CONSTANT_NameAndType,
        JVM_CONSTANT_MethodHandle           = 15,  // JSR 292
        JVM_CONSTANT_MethodType             = 16,  // JSR 292
        //JVM_CONSTANT_(unused)             = 17,  // JSR 292 early drafts only
        JVM_CONSTANT_InvokeDynamic          = 18,  // JSR 292
        JVM_CONSTANT_ExternalMax            = 18   // Last tag found in classfiles
```

代わりに, このクラスでは HotSpot に特有なタグの定義が行われている.


```cpp
    ((cite: hotspot/src/share/vm/utilities/constantTag.hpp))
      // See jvm.h for shared JVM_CONSTANT_XXX tags
      // NOTE: replicated in SA in vm/agent/sun/jvm/hotspot/utilities/ConstantTag.java
      // Hotspot specific tags
      JVM_CONSTANT_Invalid                  = 0,    // For bad value initialization
      JVM_CONSTANT_InternalMin              = 100,  // First implementation tag (aside from bad value of course)
      JVM_CONSTANT_UnresolvedClass          = 100,  // Temporary tag until actual use
      JVM_CONSTANT_ClassIndex               = 101,  // Temporary tag while constructing constant pool
      JVM_CONSTANT_UnresolvedString         = 102,  // Temporary tag until actual use
      JVM_CONSTANT_StringIndex              = 103,  // Temporary tag while constructing constant pool
      JVM_CONSTANT_UnresolvedClassInError   = 104,  // Error tag due to resolution error
      JVM_CONSTANT_Object                   = 105,  // Required for BoundMethodHandle arguments.
      JVM_CONSTANT_InternalMax              = 105   // Last implementation tag
```

また, 各タグのビットがたっているかどうかを調べるためのメソッドも提供している

(このメソッドについては標準的なタグ用のものも含めて提供されている)


```cpp
    ((cite: hotspot/src/share/vm/utilities/constantTag.hpp))
      bool is_klass() const             { return _tag == JVM_CONSTANT_Class; }
      bool is_field () const            { return _tag == JVM_CONSTANT_Fieldref; }
      bool is_method() const            { return _tag == JVM_CONSTANT_Methodref; }
      bool is_interface_method() const  { return _tag == JVM_CONSTANT_InterfaceMethodref; }
      bool is_string() const            { return _tag == JVM_CONSTANT_String; }
      bool is_int() const               { return _tag == JVM_CONSTANT_Integer; }
      bool is_float() const             { return _tag == JVM_CONSTANT_Float; }
      bool is_long() const              { return _tag == JVM_CONSTANT_Long; }
      bool is_double() const            { return _tag == JVM_CONSTANT_Double; }
      bool is_name_and_type() const     { return _tag == JVM_CONSTANT_NameAndType; }
      bool is_utf8() const              { return _tag == JVM_CONSTANT_Utf8; }
    
      bool is_invalid() const           { return _tag == JVM_CONSTANT_Invalid; }
    
      bool is_unresolved_klass() const {
        return _tag == JVM_CONSTANT_UnresolvedClass || _tag == JVM_CONSTANT_UnresolvedClassInError;
      }
    
      bool is_unresolved_klass_in_error() const {
        return _tag == JVM_CONSTANT_UnresolvedClassInError;
      }
    
      bool is_klass_index() const       { return _tag == JVM_CONSTANT_ClassIndex; }
      bool is_unresolved_string() const { return _tag == JVM_CONSTANT_UnresolvedString; }
      bool is_string_index() const      { return _tag == JVM_CONSTANT_StringIndex; }
    
      bool is_object() const            { return _tag == JVM_CONSTANT_Object; }
    
      bool is_klass_reference() const   { return is_klass_index() || is_unresolved_klass(); }
      bool is_klass_or_reference() const{ return is_klass() || is_klass_reference(); }
      bool is_field_or_method() const   { return is_field() || is_method() || is_interface_method(); }
      bool is_symbol() const            { return is_utf8(); }
    
      bool is_method_type() const              { return _tag == JVM_CONSTANT_MethodType; }
      bool is_method_handle() const            { return _tag == JVM_CONSTANT_MethodHandle; }
      bool is_invoke_dynamic() const           { return _tag == JVM_CONSTANT_InvokeDynamic; }
    
      bool is_loadable_constant() const {
        return ((_tag >= JVM_CONSTANT_Integer && _tag <= JVM_CONSTANT_String) ||
                is_method_type() || is_method_handle() ||
                is_unresolved_klass() || is_unresolved_string() ||
                is_object());
      }
```




### 詳細(Details)
See: [here](../doxygen/classconstantTag.html) for details

---
