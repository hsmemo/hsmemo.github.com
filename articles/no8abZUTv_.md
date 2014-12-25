---
layout: default
title: JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： (おまけ) 各 JNIEnv の中身
---
[Up](no5248dl2.html) [Top](../index.html)

#### JNI の処理 ： Invocation API の処理 ： GetEnv() の処理 ： (おまけ) 各 JNIEnv の中身

--- 
## 概要(Summary)
各 JNIEnv の具体的な中身は以下の通り.

### jni_NativeInterface

```cpp
    ((cite: hotspot/src/share/vm/prims/jni.cpp))
    // Structure containing all jni functions
    struct JNINativeInterface_ jni_NativeInterface = {
        NULL,
        NULL,
        NULL,
    
        NULL,
    
        jni_GetVersion,
    
        jni_DefineClass,
        jni_FindClass,
    
        jni_FromReflectedMethod,
        jni_FromReflectedField,
    
        jni_ToReflectedMethod,
    
        jni_GetSuperclass,
        jni_IsAssignableFrom,
    
        jni_ToReflectedField,
    
        jni_Throw,
        jni_ThrowNew,
        jni_ExceptionOccurred,
        jni_ExceptionDescribe,
        jni_ExceptionClear,
        jni_FatalError,
    
        jni_PushLocalFrame,
        jni_PopLocalFrame,
    
        jni_NewGlobalRef,
        jni_DeleteGlobalRef,
        jni_DeleteLocalRef,
        jni_IsSameObject,
    
        jni_NewLocalRef,
        jni_EnsureLocalCapacity,
    
        jni_AllocObject,
        jni_NewObject,
        jni_NewObjectV,
        jni_NewObjectA,
    
        jni_GetObjectClass,
        jni_IsInstanceOf,
    
        jni_GetMethodID,
    
        jni_CallObjectMethod,
        jni_CallObjectMethodV,
        jni_CallObjectMethodA,
        jni_CallBooleanMethod,
        jni_CallBooleanMethodV,
        jni_CallBooleanMethodA,
        jni_CallByteMethod,
        jni_CallByteMethodV,
        jni_CallByteMethodA,
        jni_CallCharMethod,
        jni_CallCharMethodV,
        jni_CallCharMethodA,
        jni_CallShortMethod,
        jni_CallShortMethodV,
        jni_CallShortMethodA,
        jni_CallIntMethod,
        jni_CallIntMethodV,
        jni_CallIntMethodA,
        jni_CallLongMethod,
        jni_CallLongMethodV,
        jni_CallLongMethodA,
        jni_CallFloatMethod,
        jni_CallFloatMethodV,
        jni_CallFloatMethodA,
        jni_CallDoubleMethod,
        jni_CallDoubleMethodV,
        jni_CallDoubleMethodA,
        jni_CallVoidMethod,
        jni_CallVoidMethodV,
        jni_CallVoidMethodA,
    
        jni_CallNonvirtualObjectMethod,
        jni_CallNonvirtualObjectMethodV,
        jni_CallNonvirtualObjectMethodA,
        jni_CallNonvirtualBooleanMethod,
        jni_CallNonvirtualBooleanMethodV,
        jni_CallNonvirtualBooleanMethodA,
        jni_CallNonvirtualByteMethod,
        jni_CallNonvirtualByteMethodV,
        jni_CallNonvirtualByteMethodA,
        jni_CallNonvirtualCharMethod,
        jni_CallNonvirtualCharMethodV,
        jni_CallNonvirtualCharMethodA,
        jni_CallNonvirtualShortMethod,
        jni_CallNonvirtualShortMethodV,
        jni_CallNonvirtualShortMethodA,
        jni_CallNonvirtualIntMethod,
        jni_CallNonvirtualIntMethodV,
        jni_CallNonvirtualIntMethodA,
        jni_CallNonvirtualLongMethod,
        jni_CallNonvirtualLongMethodV,
        jni_CallNonvirtualLongMethodA,
        jni_CallNonvirtualFloatMethod,
        jni_CallNonvirtualFloatMethodV,
        jni_CallNonvirtualFloatMethodA,
        jni_CallNonvirtualDoubleMethod,
        jni_CallNonvirtualDoubleMethodV,
        jni_CallNonvirtualDoubleMethodA,
        jni_CallNonvirtualVoidMethod,
        jni_CallNonvirtualVoidMethodV,
        jni_CallNonvirtualVoidMethodA,
    
        jni_GetFieldID,
    
        jni_GetObjectField,
        jni_GetBooleanField,
        jni_GetByteField,
        jni_GetCharField,
        jni_GetShortField,
        jni_GetIntField,
        jni_GetLongField,
        jni_GetFloatField,
        jni_GetDoubleField,
    
        jni_SetObjectField,
        jni_SetBooleanField,
        jni_SetByteField,
        jni_SetCharField,
        jni_SetShortField,
        jni_SetIntField,
        jni_SetLongField,
        jni_SetFloatField,
        jni_SetDoubleField,
    
        jni_GetStaticMethodID,
    
        jni_CallStaticObjectMethod,
        jni_CallStaticObjectMethodV,
        jni_CallStaticObjectMethodA,
        jni_CallStaticBooleanMethod,
        jni_CallStaticBooleanMethodV,
        jni_CallStaticBooleanMethodA,
        jni_CallStaticByteMethod,
        jni_CallStaticByteMethodV,
        jni_CallStaticByteMethodA,
        jni_CallStaticCharMethod,
        jni_CallStaticCharMethodV,
        jni_CallStaticCharMethodA,
        jni_CallStaticShortMethod,
        jni_CallStaticShortMethodV,
        jni_CallStaticShortMethodA,
        jni_CallStaticIntMethod,
        jni_CallStaticIntMethodV,
        jni_CallStaticIntMethodA,
        jni_CallStaticLongMethod,
        jni_CallStaticLongMethodV,
        jni_CallStaticLongMethodA,
        jni_CallStaticFloatMethod,
        jni_CallStaticFloatMethodV,
        jni_CallStaticFloatMethodA,
        jni_CallStaticDoubleMethod,
        jni_CallStaticDoubleMethodV,
        jni_CallStaticDoubleMethodA,
        jni_CallStaticVoidMethod,
        jni_CallStaticVoidMethodV,
        jni_CallStaticVoidMethodA,
    
        jni_GetStaticFieldID,
    
        jni_GetStaticObjectField,
        jni_GetStaticBooleanField,
        jni_GetStaticByteField,
        jni_GetStaticCharField,
        jni_GetStaticShortField,
        jni_GetStaticIntField,
        jni_GetStaticLongField,
        jni_GetStaticFloatField,
        jni_GetStaticDoubleField,
    
        jni_SetStaticObjectField,
        jni_SetStaticBooleanField,
        jni_SetStaticByteField,
        jni_SetStaticCharField,
        jni_SetStaticShortField,
        jni_SetStaticIntField,
        jni_SetStaticLongField,
        jni_SetStaticFloatField,
        jni_SetStaticDoubleField,
    
        jni_NewString,
        jni_GetStringLength,
        jni_GetStringChars,
        jni_ReleaseStringChars,
    
        jni_NewStringUTF,
        jni_GetStringUTFLength,
        jni_GetStringUTFChars,
        jni_ReleaseStringUTFChars,
    
        jni_GetArrayLength,
    
        jni_NewObjectArray,
        jni_GetObjectArrayElement,
        jni_SetObjectArrayElement,
    
        jni_NewBooleanArray,
        jni_NewByteArray,
        jni_NewCharArray,
        jni_NewShortArray,
        jni_NewIntArray,
        jni_NewLongArray,
        jni_NewFloatArray,
        jni_NewDoubleArray,
    
        jni_GetBooleanArrayElements,
        jni_GetByteArrayElements,
        jni_GetCharArrayElements,
        jni_GetShortArrayElements,
        jni_GetIntArrayElements,
        jni_GetLongArrayElements,
        jni_GetFloatArrayElements,
        jni_GetDoubleArrayElements,
    
        jni_ReleaseBooleanArrayElements,
        jni_ReleaseByteArrayElements,
        jni_ReleaseCharArrayElements,
        jni_ReleaseShortArrayElements,
        jni_ReleaseIntArrayElements,
        jni_ReleaseLongArrayElements,
        jni_ReleaseFloatArrayElements,
        jni_ReleaseDoubleArrayElements,
    
        jni_GetBooleanArrayRegion,
        jni_GetByteArrayRegion,
        jni_GetCharArrayRegion,
        jni_GetShortArrayRegion,
        jni_GetIntArrayRegion,
        jni_GetLongArrayRegion,
        jni_GetFloatArrayRegion,
        jni_GetDoubleArrayRegion,
    
        jni_SetBooleanArrayRegion,
        jni_SetByteArrayRegion,
        jni_SetCharArrayRegion,
        jni_SetShortArrayRegion,
        jni_SetIntArrayRegion,
        jni_SetLongArrayRegion,
        jni_SetFloatArrayRegion,
        jni_SetDoubleArrayRegion,
    
        jni_RegisterNatives,
        jni_UnregisterNatives,
    
        jni_MonitorEnter,
        jni_MonitorExit,
    
        jni_GetJavaVM,
    
        jni_GetStringRegion,
        jni_GetStringUTFRegion,
    
        jni_GetPrimitiveArrayCritical,
        jni_ReleasePrimitiveArrayCritical,
    
        jni_GetStringCritical,
        jni_ReleaseStringCritical,
    
        jni_NewWeakGlobalRef,
        jni_DeleteWeakGlobalRef,
    
        jni_ExceptionCheck,
    
        jni_NewDirectByteBuffer,
        jni_GetDirectBufferAddress,
        jni_GetDirectBufferCapacity,
    
        // New 1_6 features
    
        jni_GetObjectRefType
    };
```

### checked_jni_NativeInterface

```cpp
    ((cite: hotspot/src/share/vm/prims/jniCheck.cpp))
    /*
     * Structure containing all checked jni functions
     */
    struct JNINativeInterface_  checked_jni_NativeInterface = {
        NULL,
        NULL,
        NULL,
    
        NULL,
    
        checked_jni_GetVersion,
    
        checked_jni_DefineClass,
        checked_jni_FindClass,
    
        checked_jni_FromReflectedMethod,
        checked_jni_FromReflectedField,
    
        checked_jni_ToReflectedMethod,
    
        checked_jni_GetSuperclass,
        checked_jni_IsAssignableFrom,
    
        checked_jni_ToReflectedField,
    
        checked_jni_Throw,
        checked_jni_ThrowNew,
        checked_jni_ExceptionOccurred,
        checked_jni_ExceptionDescribe,
        checked_jni_ExceptionClear,
        checked_jni_FatalError,
    
        checked_jni_PushLocalFrame,
        checked_jni_PopLocalFrame,
    
        checked_jni_NewGlobalRef,
        checked_jni_DeleteGlobalRef,
        checked_jni_DeleteLocalRef,
        checked_jni_IsSameObject,
    
        checked_jni_NewLocalRef,
        checked_jni_EnsureLocalCapacity,
    
        checked_jni_AllocObject,
        checked_jni_NewObject,
        checked_jni_NewObjectV,
        checked_jni_NewObjectA,
    
        checked_jni_GetObjectClass,
        checked_jni_IsInstanceOf,
    
        checked_jni_GetMethodID,
    
        checked_jni_CallObjectMethod,
        checked_jni_CallObjectMethodV,
        checked_jni_CallObjectMethodA,
        checked_jni_CallBooleanMethod,
        checked_jni_CallBooleanMethodV,
        checked_jni_CallBooleanMethodA,
        checked_jni_CallByteMethod,
        checked_jni_CallByteMethodV,
        checked_jni_CallByteMethodA,
        checked_jni_CallCharMethod,
        checked_jni_CallCharMethodV,
        checked_jni_CallCharMethodA,
        checked_jni_CallShortMethod,
        checked_jni_CallShortMethodV,
        checked_jni_CallShortMethodA,
        checked_jni_CallIntMethod,
        checked_jni_CallIntMethodV,
        checked_jni_CallIntMethodA,
        checked_jni_CallLongMethod,
        checked_jni_CallLongMethodV,
        checked_jni_CallLongMethodA,
        checked_jni_CallFloatMethod,
        checked_jni_CallFloatMethodV,
        checked_jni_CallFloatMethodA,
        checked_jni_CallDoubleMethod,
        checked_jni_CallDoubleMethodV,
        checked_jni_CallDoubleMethodA,
        checked_jni_CallVoidMethod,
        checked_jni_CallVoidMethodV,
        checked_jni_CallVoidMethodA,
    
        checked_jni_CallNonvirtualObjectMethod,
        checked_jni_CallNonvirtualObjectMethodV,
        checked_jni_CallNonvirtualObjectMethodA,
        checked_jni_CallNonvirtualBooleanMethod,
        checked_jni_CallNonvirtualBooleanMethodV,
        checked_jni_CallNonvirtualBooleanMethodA,
        checked_jni_CallNonvirtualByteMethod,
        checked_jni_CallNonvirtualByteMethodV,
        checked_jni_CallNonvirtualByteMethodA,
        checked_jni_CallNonvirtualCharMethod,
        checked_jni_CallNonvirtualCharMethodV,
        checked_jni_CallNonvirtualCharMethodA,
        checked_jni_CallNonvirtualShortMethod,
        checked_jni_CallNonvirtualShortMethodV,
        checked_jni_CallNonvirtualShortMethodA,
        checked_jni_CallNonvirtualIntMethod,
        checked_jni_CallNonvirtualIntMethodV,
        checked_jni_CallNonvirtualIntMethodA,
        checked_jni_CallNonvirtualLongMethod,
        checked_jni_CallNonvirtualLongMethodV,
        checked_jni_CallNonvirtualLongMethodA,
        checked_jni_CallNonvirtualFloatMethod,
        checked_jni_CallNonvirtualFloatMethodV,
        checked_jni_CallNonvirtualFloatMethodA,
        checked_jni_CallNonvirtualDoubleMethod,
        checked_jni_CallNonvirtualDoubleMethodV,
        checked_jni_CallNonvirtualDoubleMethodA,
        checked_jni_CallNonvirtualVoidMethod,
        checked_jni_CallNonvirtualVoidMethodV,
        checked_jni_CallNonvirtualVoidMethodA,
    
        checked_jni_GetFieldID,
    
        checked_jni_GetObjectField,
        checked_jni_GetBooleanField,
        checked_jni_GetByteField,
        checked_jni_GetCharField,
        checked_jni_GetShortField,
        checked_jni_GetIntField,
        checked_jni_GetLongField,
        checked_jni_GetFloatField,
        checked_jni_GetDoubleField,
    
        checked_jni_SetObjectField,
        checked_jni_SetBooleanField,
        checked_jni_SetByteField,
        checked_jni_SetCharField,
        checked_jni_SetShortField,
        checked_jni_SetIntField,
        checked_jni_SetLongField,
        checked_jni_SetFloatField,
        checked_jni_SetDoubleField,
    
        checked_jni_GetStaticMethodID,
    
        checked_jni_CallStaticObjectMethod,
        checked_jni_CallStaticObjectMethodV,
        checked_jni_CallStaticObjectMethodA,
        checked_jni_CallStaticBooleanMethod,
        checked_jni_CallStaticBooleanMethodV,
        checked_jni_CallStaticBooleanMethodA,
        checked_jni_CallStaticByteMethod,
        checked_jni_CallStaticByteMethodV,
        checked_jni_CallStaticByteMethodA,
        checked_jni_CallStaticCharMethod,
        checked_jni_CallStaticCharMethodV,
        checked_jni_CallStaticCharMethodA,
        checked_jni_CallStaticShortMethod,
        checked_jni_CallStaticShortMethodV,
        checked_jni_CallStaticShortMethodA,
        checked_jni_CallStaticIntMethod,
        checked_jni_CallStaticIntMethodV,
        checked_jni_CallStaticIntMethodA,
        checked_jni_CallStaticLongMethod,
        checked_jni_CallStaticLongMethodV,
        checked_jni_CallStaticLongMethodA,
        checked_jni_CallStaticFloatMethod,
        checked_jni_CallStaticFloatMethodV,
        checked_jni_CallStaticFloatMethodA,
        checked_jni_CallStaticDoubleMethod,
        checked_jni_CallStaticDoubleMethodV,
        checked_jni_CallStaticDoubleMethodA,
        checked_jni_CallStaticVoidMethod,
        checked_jni_CallStaticVoidMethodV,
        checked_jni_CallStaticVoidMethodA,
    
        checked_jni_GetStaticFieldID,
    
        checked_jni_GetStaticObjectField,
        checked_jni_GetStaticBooleanField,
        checked_jni_GetStaticByteField,
        checked_jni_GetStaticCharField,
        checked_jni_GetStaticShortField,
        checked_jni_GetStaticIntField,
        checked_jni_GetStaticLongField,
        checked_jni_GetStaticFloatField,
        checked_jni_GetStaticDoubleField,
    
        checked_jni_SetStaticObjectField,
        checked_jni_SetStaticBooleanField,
        checked_jni_SetStaticByteField,
        checked_jni_SetStaticCharField,
        checked_jni_SetStaticShortField,
        checked_jni_SetStaticIntField,
        checked_jni_SetStaticLongField,
        checked_jni_SetStaticFloatField,
        checked_jni_SetStaticDoubleField,
    
        checked_jni_NewString,
        checked_jni_GetStringLength,
        checked_jni_GetStringChars,
        checked_jni_ReleaseStringChars,
    
        checked_jni_NewStringUTF,
        checked_jni_GetStringUTFLength,
        checked_jni_GetStringUTFChars,
        checked_jni_ReleaseStringUTFChars,
    
        checked_jni_GetArrayLength,
    
        checked_jni_NewObjectArray,
        checked_jni_GetObjectArrayElement,
        checked_jni_SetObjectArrayElement,
    
        checked_jni_NewBooleanArray,
        checked_jni_NewByteArray,
        checked_jni_NewCharArray,
        checked_jni_NewShortArray,
        checked_jni_NewIntArray,
        checked_jni_NewLongArray,
        checked_jni_NewFloatArray,
        checked_jni_NewDoubleArray,
    
        checked_jni_GetBooleanArrayElements,
        checked_jni_GetByteArrayElements,
        checked_jni_GetCharArrayElements,
        checked_jni_GetShortArrayElements,
        checked_jni_GetIntArrayElements,
        checked_jni_GetLongArrayElements,
        checked_jni_GetFloatArrayElements,
        checked_jni_GetDoubleArrayElements,
    
        checked_jni_ReleaseBooleanArrayElements,
        checked_jni_ReleaseByteArrayElements,
        checked_jni_ReleaseCharArrayElements,
        checked_jni_ReleaseShortArrayElements,
        checked_jni_ReleaseIntArrayElements,
        checked_jni_ReleaseLongArrayElements,
        checked_jni_ReleaseFloatArrayElements,
        checked_jni_ReleaseDoubleArrayElements,
    
        checked_jni_GetBooleanArrayRegion,
        checked_jni_GetByteArrayRegion,
        checked_jni_GetCharArrayRegion,
        checked_jni_GetShortArrayRegion,
        checked_jni_GetIntArrayRegion,
        checked_jni_GetLongArrayRegion,
        checked_jni_GetFloatArrayRegion,
        checked_jni_GetDoubleArrayRegion,
    
        checked_jni_SetBooleanArrayRegion,
        checked_jni_SetByteArrayRegion,
        checked_jni_SetCharArrayRegion,
        checked_jni_SetShortArrayRegion,
        checked_jni_SetIntArrayRegion,
        checked_jni_SetLongArrayRegion,
        checked_jni_SetFloatArrayRegion,
        checked_jni_SetDoubleArrayRegion,
    
        checked_jni_RegisterNatives,
        checked_jni_UnregisterNatives,
    
        checked_jni_MonitorEnter,
        checked_jni_MonitorExit,
    
        checked_jni_GetJavaVM,
    
        checked_jni_GetStringRegion,
        checked_jni_GetStringUTFRegion,
    
        checked_jni_GetPrimitiveArrayCritical,
        checked_jni_ReleasePrimitiveArrayCritical,
    
        checked_jni_GetStringCritical,
        checked_jni_ReleaseStringCritical,
    
        checked_jni_NewWeakGlobalRef,
        checked_jni_DeleteWeakGlobalRef,
    
        checked_jni_ExceptionCheck,
    
        checked_jni_NewDirectByteBuffer,
        checked_jni_GetDirectBufferAddress,
        checked_jni_GetDirectBufferCapacity,
    
        // New 1.6 Features
    
        checked_jni_GetObjectRefType
    };
```


=






