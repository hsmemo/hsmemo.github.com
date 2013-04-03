---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/classfile/classFileParser.cpp

### 名前(function name)
```
unsigned int
ClassFileParser::compute_oop_map_count(instanceKlassHandle super,
                                       unsigned int nonstatic_oop_map_count,
                                       int first_nonstatic_oop_offset) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の数の和を OopMap の大きさとしてリターンする.
      * スーパークラスの nonstatic_oop_map_count の数
      * nonstatic_oop_map_count 引数で指定された数(-1)
  
      ただし, それぞれ以下のように計算する.
      * スーパークラスの nonstatic_oop_map_count の数: 
        super 引数が NULL の場合は 0 とする. そうでなければ instanceKlass::nonstatic_oop_map_count() で取得する.
      * nonstatic_oop_map_count 引数で指定された数(-1): 
        基本的には nonstatic_oop_map_count 引数の数そのままでいい.
        ただし, このクラスの最初の oop フィールドがスーパークラスの最後の oop フィールドと連続している場合は
        それらは1つにまとめられるので 1 引いた数とする.
      ---------------------------------------- -}

	  unsigned int map_count =
	    super.is_null() ? 0 : super->nonstatic_oop_map_count();
	  if (nonstatic_oop_map_count > 0) {
	    // We have oops to add to map
	    if (map_count == 0) {
	      map_count = nonstatic_oop_map_count;
	    } else {
	      // Check whether we should add a new map block or whether the last one can
	      // be extended
	      OopMapBlock* const first_map = super->start_of_nonstatic_oop_maps();
	      OopMapBlock* const last_map = first_map + map_count - 1;
	
	      int next_offset = last_map->offset() + last_map->count() * heapOopSize;
	      if (next_offset == first_nonstatic_oop_offset) {
	        // There is no gap bettwen superklass's last oop field and first
	        // local oop field, merge maps.
	        nonstatic_oop_map_count -= 1;
	      } else {
	        // Superklass didn't end with a oop field, add extra maps
	        assert(next_offset < first_nonstatic_oop_offset, "just checking");
	      }
	      map_count += nonstatic_oop_map_count;
	    }
	  }
	  return map_count;
	}
	
```


