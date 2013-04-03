---
layout: default
title: ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： classfile/
---
[Up](nopoim3uPN.html) [Top](../index.html)

#### ソースコードのディレクトリ構成 ： hotspot/src/share/ 以下 ： classfile/

--- 

File Name                                                 | Description
--------------------------------------------------------- | -----------------------------------------------------------------
hotspot/src/share/vm/classfile/classFileError.cpp         | ClassFileParser クラスや StackMapStream クラスの一部のメソッドの定義 (※1)
hotspot/src/share/vm/classfile/classFileParser.cpp        | ClassFileParser クラスの定義 ([ClassFileParser, 及びその補助クラス(ConstantPoolCleaner, NameSigHash, Classfile_LVT_Element, LVT_Hash)](nopa5hajLq.html))
hotspot/src/share/vm/classfile/classFileParser.hpp        | 同上
hotspot/src/share/vm/classfile/classFileStream.cpp        | ClassFileStream クラスの定義 ([ClassFileStream](noWw9KWx8M.html))
hotspot/src/share/vm/classfile/classFileStream.hpp        | 同上
hotspot/src/share/vm/classfile/classLoader.cpp	          | ブートストラップ・クラスローダ関連のクラスの定義 ([MetaIndex, ClassPathEntry, ClassPathDirEntry, ClassPathZipEntry, LazyClassPathEntry, ClassLoader, PerfClassTraceTime, 及びそれらの補助クラス(PackageInfo, PackageHashtable)](novfX2TLZv.html))
hotspot/src/share/vm/classfile/classLoader.hpp	          | 同上
hotspot/src/share/vm/classfile/dictionary.cpp	          | Dictionary クラス関連のクラスの定義 ([Dictionary, ProtectionDomainEntry, DictionaryEntry, SymbolPropertyEntry, SymbolPropertyTable](no8ayfdhjK.html))
hotspot/src/share/vm/classfile/dictionary.hpp	          | 同上
hotspot/src/share/vm/classfile/javaAssertions.cpp         | JavaAssertions およびその補助クラスの定義 ([JavaAssertions, JavaAssertions::OptionList](noBP3s4-ZE.html))
hotspot/src/share/vm/classfile/javaAssertions.hpp         | 同上
hotspot/src/share/vm/classfile/javaClasses.cpp	          | 標準ライブラリ内の基本的な Java クラスを扱うためのクラス群の定義 ([java_lang_String, java_lang_Class, java_lang_Thread, java_lang_ThreadGroup, java_lang_Throwable, java_lang_reflect_AccessibleObject, java_lang_reflect_Method, java_lang_reflect_Constructor, java_lang_reflect_Field, sun_reflect_ConstantPool, sun_reflect_UnsafeStaticFieldAccessorImpl, java_lang_boxing_object, java_lang_ref_Reference, java_lang_ref_SoftReference, java_lang_invoke_MethodHandle, java_lang_invoke_DirectMethodHandle, java_lang_invoke_BoundMethodHandle, java_lang_invoke_AdapterMethodHandle, java_lang_invoke_MemberName, java_lang_invoke_MethodType, java_lang_invoke_MethodTypeForm, java_lang_invoke_CallSite, java_security_AccessControlContext, java_lang_ClassLoader, java_lang_System, java_lang_StackTraceElement, java_lang_AssertionStatusDirectives, java_nio_Buffer, sun_misc_AtomicLongCSImpl, java_util_concurrent_locks_AbstractOwnableSynchronizer, JavaClasses, 及びそれらの補助クラス(BacktraceBuilder)](noMjXcMvc2.html))
hotspot/src/share/vm/classfile/javaClasses.hpp	          | 同上
hotspot/src/share/vm/classfile/loaderConstraints.cpp      | LoaderConstraint クラス関連のクラスの定義 ([LoaderConstraintTable, LoaderConstraintEntry](noaXIba8K5.html))
hotspot/src/share/vm/classfile/loaderConstraints.hpp      | 同上
hotspot/src/share/vm/classfile/placeholders.cpp	          | Placeholder クラス関連のクラスの定義 ([PlaceholderTable, SeenThread, PlaceholderEntry](normNjc4yC.html))
hotspot/src/share/vm/classfile/placeholders.hpp	          | 同上
hotspot/src/share/vm/classfile/resolutionErrors.cpp       | ResolutionError クラス関連のクラスの定義 ([ResolutionErrorTable, ResolutionErrorEntry](nood8sbVTv.html))
hotspot/src/share/vm/classfile/resolutionErrors.hpp       | 同上
hotspot/src/share/vm/classfile/stackMapFrame.cpp          | StackMapFrame クラスの定義 ([StackMapFrame](noR94qrx_c.html))
hotspot/src/share/vm/classfile/stackMapFrame.hpp          | 同上
hotspot/src/share/vm/classfile/stackMapTable.cpp          | StackMapTable クラス関連のクラスの定義 ([StackMapTable, StackMapStream, StackMapReader](noA0o9iUjP.html))
hotspot/src/share/vm/classfile/stackMapTable.hpp          | 同上
hotspot/src/share/vm/classfile/stackMapTableFormat.hpp    | クラスファイル中の StackMap attribute のデータ構造を表したクラス群の定義 ([verification_type_info, stack_map_frame, same_frame_extended, same_frame_1_stack_item_extended, full_frame, stack_map_table_attribute, 及びそれらの補助クラス(same_frame, same_frame_1_stack_item_frame, chop_frame, append_frame)](noBPWMaLuz.html))
hotspot/src/share/vm/classfile/symbolTable.cpp	          | SymbolTable クラス関連のクラスの定義 ([TempNewSymbol, SymbolTable, StringTable, 及びそれらの補助クラス(StableMemoryChecker)](noop30IFn9.html))
hotspot/src/share/vm/classfile/symbolTable.hpp	          | 同上
hotspot/src/share/vm/classfile/systemDictionary.cpp       | systemDictionary クラス関連のクラスの定義 ([SystemDictionary, SystemDictionaryHandles, 及びそれらの補助クラス(ClassStatistics, MethodStatistics)](noN4s53zWl.html))
hotspot/src/share/vm/classfile/systemDictionary.hpp       | 同上
hotspot/src/share/vm/classfile/verificationType.cpp       | VerificationType クラスの定義 ([VerificationType](no01CvtdGM.html))
hotspot/src/share/vm/classfile/verificationType.hpp       | 同上
hotspot/src/share/vm/classfile/verifier.cpp               | Verifier クラス関連のクラスの定義 ([Verifier, ClassVerifier](noU0Fy2z_Z.html))
hotspot/src/share/vm/classfile/verifier.hpp               | 同上
hotspot/src/share/vm/classfile/vmSymbols.cpp	          | vmSymbols クラス関連のクラスの定義 ([vmSymbols, vmIntrinsics](nox_cQ3rE6.html))
hotspot/src/share/vm/classfile/vmSymbols.hpp	          | 同上

### 備考(Notes)
* ※1: inlining を避けるため別ファイルにしているとのこと

```
    ((cite: hotspot/src/share/vm/classfile/classFileError.cpp))
    // Keep these in a separate file to prevent inlining
```






