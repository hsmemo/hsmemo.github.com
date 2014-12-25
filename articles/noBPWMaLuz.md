---
layout: default
title: クラスファイル中の StackMap attribute のデータ構造を表したクラス群 (verification_type_info, stack_map_frame, same_frame_extended, same_frame_1_stack_item_extended, full_frame, stack_map_table_attribute, 及びそれらの補助クラス(same_frame, same_frame_1_stack_item_frame, chop_frame, append_frame))
---
[Top](../index.html)

#### クラスファイル中の StackMap attribute のデータ構造を表したクラス群 (verification_type_info, stack_map_frame, same_frame_extended, same_frame_1_stack_item_extended, full_frame, stack_map_table_attribute, 及びそれらの補助クラス(same_frame, same_frame_1_stack_item_frame, chop_frame, append_frame))

これらは, クラスファイル中の StackMap attribute のデータ構造を表したクラス.
なお, これらは Relocator クラス の Relocator::adjust_stack_map_table() メソッド内で(のみ)使用される補助クラス.

(クラス名も JVMS での名称に合わせている模様 (JVMS 4.7.4 参照). そのため, 他のクラスとは命名規則が異なる)


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    // These classes represent the stack-map substructures described in the JVMS
    // (hence the non-conforming naming scheme).
    
    // These classes work with the types in their compressed form in-place (as they
    // would appear in the classfile).  No virtual methods or fields allowed.
```



### クラス一覧(class list)

  * [verification_type_info](#noKYboWP6-)
  * [stack_map_frame](#noCb5Q5cf_)
  * [same_frame](#no48m7ihPR)
  * [same_frame_extended](#noTbNM45V9)
  * [same_frame_1_stack_item_frame](#no2IwoXoQh)
  * [same_frame_1_stack_item_extended](#no_ISmGDFn)
  * [chop_frame](#noVYycjdBS)
  * [append_frame](#noFIHmT5Lx)
  * [full_frame](#noMBR-MD6b)
  * [stack_map_table_attribute](#nodVVs95N4)


---
## <a name="noKYboWP6-" id="noKYboWP6-">verification_type_info</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の verification_type_info 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class verification_type_info {
```




### 詳細(Details)
See: [here](../doxygen/classverification__type__info.html) for details

---
## <a name="noCb5Q5cf_" id="noCb5Q5cf_">stack_map_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の stack_map_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class stack_map_frame {
```




### 詳細(Details)
See: [here](../doxygen/classstack__map__frame.html) for details

---
## <a name="no48m7ihPR" id="no48m7ihPR">same_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の same_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class same_frame : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classsame__frame.html) for details

---
## <a name="noTbNM45V9" id="noTbNM45V9">same_frame_extended</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の same_frame_extended 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class same_frame_extended : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classsame__frame__extended.html) for details

---
## <a name="no2IwoXoQh" id="no2IwoXoQh">same_frame_1_stack_item_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の same_frame_1_stack_item_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class same_frame_1_stack_item_frame : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classsame__frame__1__stack__item__frame.html) for details

---
## <a name="no_ISmGDFn" id="no_ISmGDFn">same_frame_1_stack_item_extended</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の same_frame_1_stack_item_extended 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class same_frame_1_stack_item_extended : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classsame__frame__1__stack__item__extended.html) for details

---
## <a name="noVYycjdBS" id="noVYycjdBS">chop_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の chop_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class chop_frame : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classchop__frame.html) for details

---
## <a name="noFIHmT5Lx" id="noFIHmT5Lx">append_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の append_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class append_frame : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classappend__frame.html) for details

---
## <a name="noMBR-MD6b" id="noMBR-MD6b">full_frame</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute の full_frame 情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class full_frame : public stack_map_frame {
```



### 詳細(Details)
See: [here](../doxygen/classfull__frame.html) for details

---
## <a name="nodVVs95N4" id="nodVVs95N4">stack_map_table_attribute</a>

### 概要(Summary)
Relocator クラス内で使用される補助クラス.

StackMap attribute のヘッダの情報を表すクラス.


```cpp
    ((cite: hotspot/src/share/vm/classfile/stackMapTableFormat.hpp))
    class stack_map_table_attribute {
```




### 詳細(Details)
See: [here](../doxygen/classstack__map__table__attribute.html) for details

---
