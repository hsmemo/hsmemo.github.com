---
layout: default
title: JNI の処理 ： JNI Functions の処理 ： ローカル参照/グローバル参照の処理(Global and Local References) ： ローカル参照フレームの処理  
---
[Up](notiXs9FLU.html) [Top](../index.html)

#### JNI の処理 ： JNI Functions の処理 ： ローカル参照/グローバル参照の処理(Global and Local References) ： ローカル参照フレームの処理  

--- 
## 該当する JNI 関数
* `EnsureLocalCapacity`,
* `PushLocalFrame`,
* `PopLocalFrame`,


## 概要(Summary)
PushLocalFrame() や PopLocalFrame() の処理は,
JNIHandleBlock の確保/開放を行って Thread::_active_handles フィールドをつなぎ直すだけ
(See: JNIHandleBlock).


## 処理の流れ (概要)(Execution Flows : Summary)
### EnsureLocalCapacity() の処理
```
jni_EnsureLocalCapacity()
```

### PushLocalFrame() の処理
```
jni_PushLocalFrame()
-> JNIHandleBlock::allocate_block()
-> JNIHandleBlock::set_pop_frame_link()
-> Thread::set_active_handles()
```

### PopLocalFrame() の処理
```
jni_PopLocalFrame()
-> Thread::set_active_handles()
-> JNIHandleBlock::set_pop_frame_link()
-> JNIHandleBlock::release_block()
```


## 処理の流れ (詳細)(Execution Flows : Details)
### jni_EnsureLocalCapacity()
See: [here](no3059CVR.html) for details

### jni_PushLocalFrame()
See: [here](no3718ePN.html) for details
### JNIHandleBlock::allocate_block()
See: [here](no3718fXI.html) for details
### JNIHandleBlock::set_pop_frame_link()
See: [here](no3059psv.html) for details
### Thread::set_active_handles()
See: [here](no3059oAF.html) for details

### jni_PopLocalFrame()
See: [here](no37184jZ.html) for details
### JNIHandleBlock::release_block()
See: [here](no30591KL.html) for details






