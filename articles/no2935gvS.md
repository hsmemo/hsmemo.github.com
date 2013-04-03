---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp

### 名前(function name)
```
jvmtiError JvmtiManageCapabilities::add_capabilities(const jvmtiCapabilities *current,
                                                     const jvmtiCapabilities *prohibited,
                                                     const jvmtiCapabilities *desired,
                                                     jvmtiCapabilities *result) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まず JvmtiManageCapabilities::get_potential_capabilities() で現在取得可能な capabilities 一覧をだし,
      取得できるのかどうかをチェックする.
      取得できないものが有ればエラーでリターン.
      (条件判定は, desired から取得可能なものを exclude して残ったbitがあるかどうかを見ている.
       もしあれば, 取得できないものがあったということ.)
      ---------------------------------------- -}

	  // check that the capabilities being added are potential capabilities
	  jvmtiCapabilities temp;
	  get_potential_capabilities(current, prohibited, &temp);
	  if (has_some(exclude(desired, &temp, &temp))) {
	    return JVMTI_ERROR_NOT_AVAILABLE;
	  }
	
  {- -------------------------------------------
  (1) acquired_capabilities に取得されたことを記録する
      ---------------------------------------- -}

	  // add to the set of ever acquired capabilities
	  either(&acquired_capabilities, desired, &acquired_capabilities);
	
  {- -------------------------------------------
  (1) 取得に成功した onload_capabilities については, always_capabilities に移動する (= いつでも取得可能とする)
      ---------------------------------------- -}

	  // onload capabilities that got added are now permanent - so, also remove from onload
	  both(&onload_capabilities, desired, &temp);
	  either(&always_capabilities, &temp, &always_capabilities);
	  exclude(&onload_capabilities, &temp, &onload_capabilities);
	
  {- -------------------------------------------
  (1) onload_solo_capabilities についても同様に処理する
      ---------------------------------------- -}

	  // same for solo capabilities (transferred capabilities in the remaining sets handled as part of standard grab - below)
	  both(&onload_solo_capabilities, desired, &temp);
	  either(&always_solo_capabilities, &temp, &always_solo_capabilities);
	  exclude(&onload_solo_capabilities, &temp, &onload_solo_capabilities);
	
  {- -------------------------------------------
  (1) 取得された solo capabilities を remaining から除去する
      ---------------------------------------- -}

	  // remove solo capabilities that are now taken
	  exclude(&always_solo_remaining_capabilities, desired, &always_solo_remaining_capabilities);
	  exclude(&onload_solo_remaining_capabilities, desired, &onload_solo_remaining_capabilities);
	
  {- -------------------------------------------
  (1) 
      ---------------------------------------- -}

	  // return the result
	  either(current, desired, result);
	
  {- -------------------------------------------
  (1) JvmtiManageCapabilities::update() を呼んで, 以上の変更を反映させる.
      (これにより, 対応する JvmtiExport::can_*() メソッドの値が変化する)
      ---------------------------------------- -}

	  update();
	
  {- -------------------------------------------
  (1) リターン
      ---------------------------------------- -}

	  return JVMTI_ERROR_NONE;
	}
	
```


