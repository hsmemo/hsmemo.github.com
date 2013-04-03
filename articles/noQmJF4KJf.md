---
layout: default
title: デバッグ情報関連のクラス (ScopeValue, LocationValue, ObjectValue, ConstantIntValue, ConstantLongValue, ConstantDoubleValue, ConstantOopWriteValue, ConstantOopReadValue, MonitorValue, DebugInfoReadStream, DebugInfoWriteStream)
---
[Top](../index.html)

#### デバッグ情報関連のクラス (ScopeValue, LocationValue, ObjectValue, ConstantIntValue, ConstantLongValue, ConstantDoubleValue, ConstantOopWriteValue, ConstantOopReadValue, MonitorValue, DebugInfoReadStream, DebugInfoWriteStream)

#Under Construction


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // Classes used for serializing debugging information.
    // These abstractions are introducted to provide symmetric
    // read and write operations.
    
    // ScopeValue        describes the value of a variable/expression in a scope
    // - LocationValue   describes a value in a given location (in frame or register)
    // - ConstantValue   describes a constant
```



### クラス一覧(class list)

  * [ScopeValue](#no63r1kb6g)
  * [LocationValue](#nokr26IY53)
  * [ObjectValue](#noenB9z0rc)
  * [ConstantIntValue](#nos5PKW3FE)
  * [ConstantLongValue](#noTL-8_rMr)
  * [ConstantDoubleValue](#nojXA6AcV9)
  * [ConstantOopWriteValue](#noD_huiA44)
  * [ConstantOopReadValue](#nokpjRweao)
  * [MonitorValue](#noWKH7MoKJ)
  * [DebugInfoReadStream](#no8gtucLQ8)
  * [DebugInfoWriteStream](#noymPq6txE)


---
## <a name="no63r1kb6g" id="no63r1kb6g">ScopeValue</a>

### 概要(Summary)
(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    class ScopeValue: public ResourceObj {
```



### 詳細(Details)
See: [here](../doxygen/classScopeValue.html) for details

---
## <a name="nokr26IY53" id="nokr26IY53">LocationValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // A Location value describes a value in a given location; i.e. the corresponding
    // logical entity (e.g., a method temporary) lives in this location.
    
    class LocationValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classLocationValue.html) for details

---
## <a name="noenB9z0rc" id="noenB9z0rc">ObjectValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // An ObjectValue describes an object eliminated by escape analysis.
    
    class ObjectValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classObjectValue.html) for details

---
## <a name="nos5PKW3FE" id="nos5PKW3FE">ConstantIntValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // A ConstantIntValue describes a constant int; i.e., the corresponding logical entity
    // is either a source constant or its computation has been constant-folded.
    
    class ConstantIntValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classConstantIntValue.html) for details

---
## <a name="noTL-8_rMr" id="noTL-8_rMr">ConstantLongValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    class ConstantLongValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classConstantLongValue.html) for details

---
## <a name="nojXA6AcV9" id="nojXA6AcV9">ConstantDoubleValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    class ConstantDoubleValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classConstantDoubleValue.html) for details

---
## <a name="noD_huiA44" id="noD_huiA44">ConstantOopWriteValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // A ConstantOopWriteValue is created by the compiler to
    // be written as debugging information.
    
    class ConstantOopWriteValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classConstantOopWriteValue.html) for details

---
## <a name="nokpjRweao" id="nokpjRweao">ConstantOopReadValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // A ConstantOopReadValue is created by the VM when reading
    // debug information
    
    class ConstantOopReadValue: public ScopeValue {
```



### 詳細(Details)
See: [here](../doxygen/classConstantOopReadValue.html) for details

---
## <a name="noWKH7MoKJ" id="noWKH7MoKJ">MonitorValue</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // MonitorValue describes the pair used for monitor_enter and monitor_exit.
    
    class MonitorValue: public ResourceObj {
```



### 詳細(Details)
See: [here](../doxygen/classMonitorValue.html) for details

---
## <a name="no8gtucLQ8" id="no8gtucLQ8">DebugInfoReadStream</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // DebugInfoReadStream specializes CompressedReadStream for reading
    // debugging information. Used by ScopeDesc.
    
    class DebugInfoReadStream : public CompressedReadStream {
```



### 詳細(Details)
See: [here](../doxygen/classDebugInfoReadStream.html) for details

---
## <a name="noymPq6txE" id="noymPq6txE">DebugInfoWriteStream</a>

(#Under Construction)


```
    ((cite: hotspot/src/share/vm/code/debugInfo.hpp))
    // DebugInfoWriteStream specializes CompressedWriteStream for
    // writing debugging information. Used by ScopeDescRecorder.
    
    class DebugInfoWriteStream : public CompressedWriteStream {
```





### 詳細(Details)
See: [here](../doxygen/classDebugInfoWriteStream.html) for details

---
