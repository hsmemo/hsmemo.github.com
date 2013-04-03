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
void JvmtiManageCapabilities::relinquish_capabilities(const jvmtiCapabilities *current,
                                                      const jvmtiCapabilities *unwanted,
                                                      jvmtiCapabilities *result) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  jvmtiCapabilities to_trash;
	  jvmtiCapabilities temp;
	
  {- -------------------------------------------
  (1) 削除が指定されたものと現在本当に持っている capabilities の和集合を計算し,
      solo のものについては, この和集合に入っていれば, (返却されたので) remaining に復活させる.
      ---------------------------------------- -}

	  // can't give up what you don't have
	  both(current, unwanted, &to_trash);
	
	  // restore solo capabilities but only those that belong
	  either(&always_solo_remaining_capabilities, both(&always_solo_capabilities, &to_trash, &temp),
	         &always_solo_remaining_capabilities);
	  either(&onload_solo_remaining_capabilities, both(&onload_solo_capabilities, &to_trash, &temp),
	         &onload_solo_remaining_capabilities);
	
  {- -------------------------------------------
  (1) JvmtiManageCapabilities::update() を呼んで, 以上の変更を反映させる.
      (これにより, 対応する JvmtiExport::can_*() メソッドの値が変化する)
      ---------------------------------------- -}

	  update();
	
  {- -------------------------------------------
  (1) current から unwanted を削除した結果を返す.
      ---------------------------------------- -}

	  // return the result
	  exclude(current, unwanted, result);
	}
	
```


