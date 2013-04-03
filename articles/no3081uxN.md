---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/objArrayKlass.cpp

### 名前(function name)
```
oop objArrayKlass::multi_allocate(int rank, jint* sizes, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int length = *sizes;
	  // Call to lower_dimension uses this pointer, so most be called before a
	  // possible GC
	  KlassHandle h_lower_dimension(THREAD, lower_dimension());

  {- -------------------------------------------
  (1) objArrayKlass::allocate() を呼んで, メモリを確保する
      ---------------------------------------- -}

	  // If length < 0 allocate will throw an exception.
	  objArrayOop array = allocate(length, CHECK_NULL);

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(array->is_parsable(), "Don't handlize unless parsable");

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  objArrayHandle h_array (THREAD, array);

  {- -------------------------------------------
  (1) もし確保する多次元配列の次元がまだ残っているなら, 
      今回確保した次元の長さ(= length)に応じて以下のどちらかを行う.
  
      * 今回確保した次元の長さ(= length)が 0 ではない場合:
        この先の次元用のメモリを確保する必要があるので, 
        objArrayKlass::multi_allocate() (または typeArrayKlass::multi_allocate()) を再帰呼び出しする.
  
      * 今回確保した次元の長さ(= length)が 0 の場合:
        0 なのでこの先の次元用のメモリを確保する必要は無い.
  
        ただし, 長さが負値になっている次元が 1つでもあった場合は例外を出さなければいけない.
        そのため, この先の次元の長さを全てチェックし, 
        長さが負値の次元があれば NegativeArraySizeException を出す.
        (そういった次元がなければ, 何もしない).
      ---------------------------------------- -}

	  if (rank > 1) {
	    if (length != 0) {
	      for (int index = 0; index < length; index++) {
	        arrayKlass* ak = arrayKlass::cast(h_lower_dimension());
	        oop sub_array = ak->multi_allocate(rank-1, &sizes[1], CHECK_NULL);
	        assert(sub_array->is_parsable(), "Don't publish until parsable");
	        h_array->obj_at_put(index, sub_array);
	      }
	    } else {
	      // Since this array dimension has zero length, nothing will be
	      // allocated, however the lower dimension values must be checked
	      // for illegal values.
	      for (int i = 0; i < rank - 1; ++i) {
	        sizes += 1;
	        if (*sizes < 0) {
	          THROW_0(vmSymbols::java_lang_NegativeArraySizeException());
	        }
	      }
	    }
	  }

  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  return h_array();
	}
	
```


