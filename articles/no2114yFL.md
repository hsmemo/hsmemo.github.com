---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psPermGen.cpp

### 名前(function name)
```
void PSPermGen::precompact() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) コンパクションによって start array (offset table) 内の情報も古くなるため, 
      ObjectStartArray::reset() で start array の中身を全てクリアしておく.
      (この後, PSMarkSweepDecorator::precompact() の中で新しい情報が書き込まれる)
      ---------------------------------------- -}

	  // Reset start array first.
	  _start_array.reset();

  {- -------------------------------------------
  (1) PSMarkSweepDecorator::precompact() を呼んで, 
      Perm 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
      ---------------------------------------- -}

	  object_mark_sweep()->precompact();
	}
	
```


