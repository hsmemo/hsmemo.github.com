---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/services/management.cpp
### 説明(description)

```
// Sets the threshold of a given memory pool.
// Returns the previous threshold.
//
// Input parameters:
//   pool      - the MemoryPoolMXBean object
//   type      - threshold type
//   threshold - the new threshold (must not be negative)
//
```

### 名前(function name)
```
JVM_ENTRY(jlong, jmm_SetPoolThreshold(JNIEnv* env, jobject obj, jmmThresholdType type, jlong threshold))
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 指定された閾値が負値の場合は, IllegalArgumentException.
      指定された閾値が max_uintx を超えている場合は, IllegalArgumentException.
    
      (なお, max_uintx は uintptr_t 型の最大値であって, unsigned int の最大値ではないことに注意.
       32bit では 2^32-1, 64bit では 2^64-1 の値になる.)
      ---------------------------------------- -}

	  if (threshold < 0) {
	    THROW_MSG_(vmSymbols::java_lang_IllegalArgumentException(),
	               "Invalid threshold value",
	               -1);
	  }
	
	  if ((size_t)threshold > max_uintx) {
	    stringStream st;
	    st.print("Invalid valid threshold value. Threshold value (" UINT64_FORMAT ") > max value of size_t (" SIZE_FORMAT ")", (size_t)threshold, max_uintx);
	    THROW_MSG_(vmSymbols::java_lang_IllegalArgumentException(), st.as_string(), -1);
	  }
	
  {- -------------------------------------------
  (1) 引数で指定された MemoryPool に対して, 
      ThresholdSupport::set_high_threshold() または ThresholdSupport::set_low_threshold() で
      指定された種別の閾値を変更する.
  
      ただし, その MemoryPool が, 指定された種別の閾値に対応していない場合には
      (ThresholdSupport::is_high_threshold_supported() または 
       ThresholdSupport::is_low_threshold_supported() が false を返す場合), 
      -1 をリターンして終了.
  
      また, 変更するのが usage threshould の場合は, 以前の閾値を覚えておく (以下の prev).
      (collection usage threshould の場合は, prev の値は 0)
      ---------------------------------------- -}

	  MemoryPool* pool = get_memory_pool_from_jobject(obj, CHECK_(0L));
	  assert(pool != NULL, "MemoryPool should exist");
	
	  jlong prev = 0;
	  switch (type) {
	    case JMM_USAGE_THRESHOLD_HIGH:
	      if (!pool->usage_threshold()->is_high_threshold_supported()) {
	        return -1;
	      }
	      prev = pool->usage_threshold()->set_high_threshold((size_t) threshold);
	      break;
	
	    case JMM_USAGE_THRESHOLD_LOW:
	      if (!pool->usage_threshold()->is_low_threshold_supported()) {
	        return -1;
	      }
	      prev = pool->usage_threshold()->set_low_threshold((size_t) threshold);
	      break;
	
	    case JMM_COLLECTION_USAGE_THRESHOLD_HIGH:
	      if (!pool->gc_usage_threshold()->is_high_threshold_supported()) {
	        return -1;
	      }
	      // return and the new threshold is effective for the next GC
	      return pool->gc_usage_threshold()->set_high_threshold((size_t) threshold);
	
	    case JMM_COLLECTION_USAGE_THRESHOLD_LOW:
	      if (!pool->gc_usage_threshold()->is_low_threshold_supported()) {
	        return -1;
	      }
	      // return and the new threshold is effective for the next GC
	      return pool->gc_usage_threshold()->set_low_threshold((size_t) threshold);
	
	    default:
	      assert(false, "Unrecognized type");
	      return -1;
	  }
	
  {- -------------------------------------------
  (1) もし閾値の設定が変更された場合には, 
      LowMemoryDetector::recompute_enabled_for_collected_pools() で 
      _enabled_for_collected_pools フィールドの値を更新し, 
      さらに LowMemoryDetector::detect_low_memory() で
      (新しい閾値を超過していれば) 通知処理を行っておく.
      ---------------------------------------- -}

	  // When the threshold is changed, reevaluate if the low memory
	  // detection is enabled.
	  if (prev != threshold) {
	    LowMemoryDetector::recompute_enabled_for_collected_pools();
	    LowMemoryDetector::detect_low_memory(pool);
	  }

  {- -------------------------------------------
  (1) prev の値をリターン
      ---------------------------------------- -}

	  return prev;
	JVM_END
	
```


