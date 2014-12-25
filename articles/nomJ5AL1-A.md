---
layout: default
title: コマンドラインオプション関連のクラス (FlagSetting, CounterSetting, IntFlagSetting, DoubleFlagSetting, CommandLineFlags, CommandLineFlagsEx)
---
[Top](../index.html)

#### コマンドラインオプション関連のクラス (FlagSetting, CounterSetting, IntFlagSetting, DoubleFlagSetting, CommandLineFlags, CommandLineFlagsEx)

これらは, HotSpot のコマンドラインオプションの値を参照/変更するためのクラス (See: [here](no7882y_Y.html) for details).


### クラス一覧(class list)

  * [FlagSetting](#no2PTd288y)
  * [CounterSetting](#noH5-mDNoP)
  * [IntFlagSetting](#no-bVToxWN)
  * [DoubleFlagSetting](#nooBlyeAJM)
  * [CommandLineFlags](#nohmAv0r6U)
  * [CommandLineFlagsEx](#nodOe8_SU6)


---
## <a name="no2PTd288y" id="no2PTd288y">FlagSetting</a>

### 概要(Summary)
ソースコード中のあるスコープの間だけ, コマンドラインオプション(Flag)の値を変更するためのユーティリティ・クラス.

このクラスは, bool 値を取る(つまり+/-での切り替えのみの)コマンドラインオプション用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
    // use FlagSetting to temporarily change some debug flag
    // e.g. FlagSetting fs(DebugThisAndThat, true);
    // restored to previous value upon leaving scope
    class FlagSetting {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

(主にデバッグ用(開発時用)に使われている模様)

(たまにコマンドラインオプションじゃないものに使われていたりするが...
 (See: Universe::genesis(), GenCollectedHeap::do_collection()))

### 内部構造(Internal structure)
コンストラクタで現在の値を記録しながら新しい値に変更し, デストラクタで元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      FlagSetting(bool& fl, bool newValue) { flag = &fl; val = fl; fl = newValue; }
      ~FlagSetting()                       { *flag = val; }
```




### 詳細(Details)
See: [here](../doxygen/classFlagSetting.html) for details

---
## <a name="noH5-mDNoP" id="noH5-mDNoP">CounterSetting</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

(指定されたカウンタ値を一時的に増加させるためのクラス??)


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
    class CounterSetting {
```

### 内部構造(Internal structure)
コンストラクタ内で値をインクリメントし, デストラクタで元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      CounterSetting(intx* cnt) { counter = cnt; (*counter)++; }
      ~CounterSetting()         { (*counter)--; }
```




### 詳細(Details)
See: [here](../doxygen/classCounterSetting.html) for details

---
## <a name="no-bVToxWN" id="no-bVToxWN">IntFlagSetting</a>

### 概要(Summary)
ソースコード中のあるスコープの間だけ, コマンドラインオプション(Flag)の値を変更するためのユーティリティ・クラス.

このクラスは, int 値を取るコマンドラインオプション用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
    class IntFlagSetting {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* GenCollectorPolicy::satisfy_failed_allocation()
* PSMarkSweep::invoke()

### 内部構造(Internal structure)
コンストラクタで現在の値を記録しながら新しい値に変更し, デストラクタで元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      IntFlagSetting(intx& fl, intx newValue) { flag = &fl; val = fl; fl = newValue; }
      ~IntFlagSetting()                       { *flag = val; }
```




### 詳細(Details)
See: [here](../doxygen/classIntFlagSetting.html) for details

---
## <a name="nooBlyeAJM" id="nooBlyeAJM">DoubleFlagSetting</a>

### 概要(Summary)
?? (このクラスは使用箇所が見当たらない...)

ソースコード中のあるスコープの間だけ, コマンドラインオプション(Flag)の値を変更するためのユーティリティ・クラス.

このクラスは, double 値を取るコマンドラインオプション用.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
    class DoubleFlagSetting {
```

### 内部構造(Internal structure)
コンストラクタで現在の値を記録しながら新しい値に変更し, デストラクタで元に戻している.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      DoubleFlagSetting(double& fl, double newValue) { flag = &fl; val = fl; fl = newValue; }
      ~DoubleFlagSetting()                           { *flag = val; }
```




### 詳細(Details)
See: [here](../doxygen/classDoubleFlagSetting.html) for details

---
## <a name="nohmAv0r6U" id="nohmAv0r6U">CommandLineFlags</a>

### 概要(Summary)
コマンドラインオプションの値を参照/変更するためのクラス
(より正確には, そのための機能を納めた名前空間. このクラスは AllStatic ではないが, static なメソッドしか持たない).


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
    class CommandLineFlags {
```

### 使われ方(Usage)
以下の箇所で(のみ)使用されている.

* コマンドラインオプションのパース処理 (より正確には, パースした値を同名の大域変数にセットする処理)
  
```
  Arguments::process_argument()
  -> Arguments::parse_argument()
     -> set_bool_flag()
     -> set_fp_numeric_flag()
     -> set_numeric_flag()
     -> set_string_flag()
     -> append_to_string_flag()
```


```cpp
    ((cite: hotspot/src/share/vm/runtime/arguments.cpp))
    static bool set_bool_flag(char* name, bool value, FlagValueOrigin origin) {
      return CommandLineFlags::boolAtPut(name, &value, origin);
    }
    
    static bool set_fp_numeric_flag(char* name, char* value, FlagValueOrigin origin) {
    ...
      if (CommandLineFlags::doubleAtPut(name, &v, origin)) {
    ...
    }
    
    static bool set_numeric_flag(char* name, char* value, FlagValueOrigin origin) {
    ...
        if (!CommandLineFlags::intxAt(name, &intx_v)) {
    ...
      if (CommandLineFlags::intxAtPut(name, &intx_v, origin)) {
    ...
      if (!is_neg && CommandLineFlags::uintxAtPut(name, &uintx_v, origin)) {
    ...
      if (!is_neg && CommandLineFlags::uint64_tAtPut(name, &uint64_t_v, origin)) {
    ...
    }
    
    static bool set_string_flag(char* name, const char* value, FlagValueOrigin origin) {
      if (!CommandLineFlags::ccstrAtPut(name, &value, origin))  return false;
    ...
    }
    
    static bool append_to_string_flag(char* name, const char* new_value, FlagValueOrigin origin) {
    ...
      if (!CommandLineFlags::ccstrAt(name, &old_value))  return false;
    ...
      (void) CommandLineFlags::ccstrAtPut(name, &value, origin);
    ...
    }
```

* Attach API の setflag コマンド

```
  set_flag()
  -> set_bool_flag()
  -> set_intx_flag()
  -> set_uintx_flag()
  -> set_uint64_t_flag()
  -> set_ccstr_flag()
```


```cpp
    ((cite: hotspot/src/share/vm/services/attachListener.cpp))
    // set a boolean global flag using value from AttachOperation
    static jint set_bool_flag(const char* name, AttachOperation* op, outputStream* out) {
    ...
      bool res = CommandLineFlags::boolAtPut((char*)name, &value, ATTACH_ON_DEMAND);
    ...
    }
    
    // set a intx global flag using value from AttachOperation
    static jint set_intx_flag(const char* name, AttachOperation* op, outputStream* out) {
    ...
      bool res = CommandLineFlags::intxAtPut((char*)name, &value, ATTACH_ON_DEMAND);
    ...
    }
    
    // set a uintx global flag using value from AttachOperation
    static jint set_uintx_flag(const char* name, AttachOperation* op, outputStream* out) {
    ...
      bool res = CommandLineFlags::uintxAtPut((char*)name, &value, ATTACH_ON_DEMAND);
    ...
    }
    
    // set a uint64_t global flag using value from AttachOperation
    static jint set_uint64_t_flag(const char* name, AttachOperation* op, outputStream* out) {
    ...
      bool res = CommandLineFlags::uint64_tAtPut((char*)name, &value, ATTACH_ON_DEMAND);
    ...
    }
    
    // set a string global flag using value from AttachOperation
    static jint set_ccstr_flag(const char* name, AttachOperation* op, outputStream* out) {
    ...
      bool res = CommandLineFlags::ccstrAtPut((char*)name, &value, ATTACH_ON_DEMAND);
    ...
    }
```

* JMM の処理 (sun.management.MemoryImpl.setVerboseGC(), sun.management.ClassLoadingImpl.setVerboseClass() の処理)

```
  jmm_SetBoolAttribute() (JMM_VERBOSE_GC を引数として呼び出される)
  -> MemoryService::set_verbose()

  jmm_SetBoolAttribute() (JMM_VERBOSE_CLASS を引数として呼び出される)
  -> ClassLoadingService::set_verbose()
```


```cpp
    ((cite: hotspot/src/share/vm/services/classLoadingService.cpp))
    bool ClassLoadingService::set_verbose(bool verbose) {
    ...
      bool succeed = CommandLineFlags::boolAtPut((char*)"TraceClassLoading", &verbose, MANAGEMENT);
    ...
    }
    
    // Caller to this function must own Management_lock
    void ClassLoadingService::reset_trace_class_unloading() {
    ...
      bool succeed = CommandLineFlags::boolAtPut((char*)"TraceClassUnloading", &value, MANAGEMENT);
    ...
    }
```


```cpp
    ((cite: hotspot/src/share/vm/services/memoryService.cpp))
    bool MemoryService::set_verbose(bool verbose) {
    ...
      bool succeed = CommandLineFlags::boolAtPut((char*)"PrintGC", &verbose, MANAGEMENT);
    ...
    }
```

* sun.management.Flag クラスの処理 
  (sun.management.Flag.setLongValue(), sun.management.Flag.setBooleanValue(), sun.management.Flag.setStringValue() の処理)
  
```
  Java_sun_management_Flag_setLongValue()
  -> jmm_SetVMGlobal()

  Java_sun_management_Flag_setBooleanValue()
  -> jmm_SetVMGlobal()

  Java_sun_management_Flag_setStringValue()
  -> jmm_SetVMGlobal()
```


```cpp
    ((cite: hotspot/src/share/vm/services/management.cpp))
    JVM_ENTRY(void, jmm_SetVMGlobal(JNIEnv *env, jstring flag_name, jvalue new_value))
    ...
        succeed = CommandLineFlags::boolAtPut(name, &bvalue, MANAGEMENT);
    ...
        succeed = CommandLineFlags::intxAtPut(name, &ivalue, MANAGEMENT);
    ...
        succeed = CommandLineFlags::uintxAtPut(name, &uvalue, MANAGEMENT);
    ...
        succeed = CommandLineFlags::uint64_tAtPut(name, &uvalue, MANAGEMENT);
    ...
        succeed = CommandLineFlags::ccstrAtPut(name, &svalue, MANAGEMENT);
```

* DTrace 関連のコマンドラインオプションの値を実行中に変える処理
  
```
  DTrace::enable_dprobes()
  -> set_bool_flag()

  DTrace::disable_dprobes()
  -> set_bool_flag()

  DTrace::set_extended_dprobes()
  -> set_bool_flag()
  
  DTrace::set_monitor_dprobes()
  -> set_bool_flag()
```


```cpp
    ((cite: hotspot/src/share/vm/services/dtraceAttacher.cpp))
    static void set_bool_flag(const char* flag, bool value) {
      CommandLineFlags::boolAtPut((char*)flag, strlen(flag), &value,
                                  ATTACH_ON_DEMAND);
    }
```

* ?? (デバッグ用(開発時用)の機能である模様. 外部から値を参照する/変更するための機能?)

```
  ?? (使用箇所が見当たらない)
  -> JVM_AccessVMBooleanFlag()

  ?? (使用箇所が見当たらない)
  -> JVM_AccessVMIntFlag()
```


```cpp
    ((cite: hotspot/src/share/vm/prims/jvm.cpp))
    #ifndef PRODUCT
    ...
    JVM_LEAF(jboolean, JVM_AccessVMBooleanFlag(const char* name, jboolean* value, jboolean is_get))
      JVMWrapper("JVM_AccessBoolVMFlag");
      return is_get ? CommandLineFlags::boolAt((char*) name, (bool*) value) : CommandLineFlags::boolAtPut((char*) name, (bool*) value, INTERNAL);
    JVM_END
    
    JVM_LEAF(jboolean, JVM_AccessVMIntFlag(const char* name, jint* value, jboolean is_get))
      JVMWrapper("JVM_AccessVMIntFlag");
      intx v;
      jboolean result = is_get ? CommandLineFlags::intxAt((char*) name, &v) : CommandLineFlags::intxAtPut((char*) name, &v, INTERNAL);
      *value = (jint)v;
      return result;
    JVM_END
```

### 内部構造(Internal structure)
内部には, 以下のメソッド(のみ)が定義されている.

(ほとんどは「引数で指定されたコマンドラインオプションに対して, その値を取得/変更する」というメソッド)


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals.hpp))
      static bool boolAt(char* name, size_t len, bool* value);
      static bool boolAt(char* name, bool* value)      { return boolAt(name, strlen(name), value); }
      static bool boolAtPut(char* name, size_t len, bool* value, FlagValueOrigin origin);
      static bool boolAtPut(char* name, bool* value, FlagValueOrigin origin)   { return boolAtPut(name, strlen(name), value, origin); }
    
      static bool intxAt(char* name, size_t len, intx* value);
      static bool intxAt(char* name, intx* value)      { return intxAt(name, strlen(name), value); }
      static bool intxAtPut(char* name, size_t len, intx* value, FlagValueOrigin origin);
      static bool intxAtPut(char* name, intx* value, FlagValueOrigin origin)   { return intxAtPut(name, strlen(name), value, origin); }
    
      static bool uintxAt(char* name, size_t len, uintx* value);
      static bool uintxAt(char* name, uintx* value)    { return uintxAt(name, strlen(name), value); }
      static bool uintxAtPut(char* name, size_t len, uintx* value, FlagValueOrigin origin);
      static bool uintxAtPut(char* name, uintx* value, FlagValueOrigin origin) { return uintxAtPut(name, strlen(name), value, origin); }
    
      static bool uint64_tAt(char* name, size_t len, uint64_t* value);
      static bool uint64_tAt(char* name, uint64_t* value) { return uint64_tAt(name, strlen(name), value); }
      static bool uint64_tAtPut(char* name, size_t len, uint64_t* value, FlagValueOrigin origin);
      static bool uint64_tAtPut(char* name, uint64_t* value, FlagValueOrigin origin) { return uint64_tAtPut(name, strlen(name), value, origin); }
    
      static bool doubleAt(char* name, size_t len, double* value);
      static bool doubleAt(char* name, double* value)    { return doubleAt(name, strlen(name), value); }
      static bool doubleAtPut(char* name, size_t len, double* value, FlagValueOrigin origin);
      static bool doubleAtPut(char* name, double* value, FlagValueOrigin origin) { return doubleAtPut(name, strlen(name), value, origin); }
    
      static bool ccstrAt(char* name, size_t len, ccstr* value);
      static bool ccstrAt(char* name, ccstr* value)    { return ccstrAt(name, strlen(name), value); }
      static bool ccstrAtPut(char* name, size_t len, ccstr* value, FlagValueOrigin origin);
      static bool ccstrAtPut(char* name, ccstr* value, FlagValueOrigin origin) { return ccstrAtPut(name, strlen(name), value, origin); }
    
      // Returns false if name is not a command line flag.
      static bool wasSetOnCmdline(const char* name, bool* value);
      static void printSetFlags();
    
      static void printFlags(bool withComments = false );
    
      static void verify() PRODUCT_RETURN;
```




### 詳細(Details)
See: [here](../doxygen/classCommandLineFlags.html) for details

---
## <a name="nodOe8_SU6" id="nodOe8_SU6">CommandLineFlagsEx</a>

### 概要(Summary)
enum 定義の再帰定義を避けるために CommandLineFlags クラスに入れられなかったメソッドを納めた名前空間
(このクラスは AllStatic ではないが, static なメソッドしか持たない).

役割としては CommandLineFlags クラスと同じ.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals_extension.hpp))
    // Can't put the following in CommandLineFlags because
    // of a circular dependency on the enum definition.
    class CommandLineFlagsEx : CommandLineFlags {
```

### 使われ方(Usage)
HotSpot 内の様々な箇所で使用されている (#TODO).

なお, 以下のように CommandLineFlagsEx クラスのメソッドを呼び出すマクロが用意されており, 
ソースコード上ではこのマクロの形で(のみ)使用されている.


```cpp
    ((cite: hotspot/src/share/vm/runtime/globals_extension.hpp))
    #define FLAG_IS_DEFAULT(name)         (CommandLineFlagsEx::is_default(FLAG_MEMBER(name)))
    #define FLAG_IS_ERGO(name)            (CommandLineFlagsEx::is_ergo(FLAG_MEMBER(name)))
    #define FLAG_IS_CMDLINE(name)         (CommandLineFlagsEx::is_cmdline(FLAG_MEMBER(name)))
    
    #define FLAG_SET_DEFAULT(name, value) ((name) = (value))
    
    #define FLAG_SET_CMDLINE(type, name, value) (CommandLineFlagsEx::type##AtPut(FLAG_MEMBER_WITH_TYPE(name,type), (type)(value), COMMAND_LINE))
    #define FLAG_SET_ERGO(type, name, value)    (CommandLineFlagsEx::type##AtPut(FLAG_MEMBER_WITH_TYPE(name,type), (type)(value), ERGONOMIC))
```




### 詳細(Details)
See: [here](../doxygen/classCommandLineFlagsEx.html) for details

---
