---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/gc_implementation/parallelScavenge/psOldGen.cpp

### 名前(function name)
```
void PSOldGen::precompact() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  ParallelScavengeHeap* heap = (ParallelScavengeHeap*)Universe::heap();
	  assert(heap->kind() == CollectedHeap::ParallelScavengeHeap, "Sanity");
	
  {- -------------------------------------------
  (1) コンパクションによって start array (offset table) 内の情報も古くなるため, 
      ObjectStartArray::reset() で start array の中身を全てクリアしておく.
      (この後, PSMarkSweepDecorator::precompact() の中で新しい情報が書き込まれる)
      ---------------------------------------- -}

	  // Reset start array first.
	  start_array()->reset();
	
  {- -------------------------------------------
  (1) PSMarkSweepDecorator::precompact() を呼んで, 
      Old 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
      ---------------------------------------- -}

	  object_mark_sweep()->precompact();
	
  {- -------------------------------------------
  (1) PSYoungGen::precompact() を呼び出して, 
      New 領域内のオブジェクトに対して
      コンパクション後のアドレスを指す forwarding pointer を埋め込む.
      ---------------------------------------- -}

	  // Now compact the young gen
	  heap->young_gen()->precompact();
	}
	
```


