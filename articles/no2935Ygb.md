---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/jfieldIDWorkaround.hpp

### 名前(function name)
```
  static jfieldID to_instance_jfieldID(klassOop k, int offset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) リターンする値を計算する.
  
      (offset 引数の下位 large_offset_mask 分の bit (= 30bits) を offset_shift 分だけ左シフトし (= 2bits 左シフト), 
       空いた下位ビットに instance_mask_in_place を足し込んだものを返値とする.)
  
      (なお, VerifyJNIFields オプションが指定されている場合は, さらに encode_klass_hash() の返値も足し込まれる.)
      ---------------------------------------- -}

	    intptr_t as_uint = ((offset & large_offset_mask) << offset_shift) | instance_mask_in_place;
	    if (VerifyJNIFields) {
	      as_uint |= encode_klass_hash(k, offset);
	    }
	    jfieldID result = (jfieldID) as_uint;

  {- -------------------------------------------
  (1) (verify)
      ---------------------------------------- -}

	#ifndef ASSERT
	    // always verify in debug mode; switchable in anything else
	    if (VerifyJNIFields)
	#endif // ASSERT
	    {
	      verify_instance_jfieldID(k, result);
	    }

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	    assert(raw_instance_offset(result) == (offset & large_offset_mask), "extract right offset");

  {- -------------------------------------------
  (1) 結果をリターンする
      ---------------------------------------- -}

	    return result;
	  }
	
```


