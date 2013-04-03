---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/prims/jvmtiImpl.cpp

### 名前(function name)
```
void GrowableCache::recache() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  int len = _elements->length();
	
  {- -------------------------------------------
  (1) _cache フィールドのメモリを開放し, 再度現在の長さ(+1)で確保し直す.
      ---------------------------------------- -}

	  FREE_C_HEAP_ARRAY(address, _cache);
	  _cache = NEW_C_HEAP_ARRAY(address,len+1);
	
  {- -------------------------------------------
  (1) _elements 中の各要素に対して 
      GrowableElement::getCacheValue() (をサブクラスがオーバーライドしたもの) を呼び出し, 
      その結果を _cache 内に詰めていく.
    
      (なお, getCacheValue() の返値として NULL を返す要素があった場合, 
       GrowableCache::remove() を呼んでその要素を除去している (このパスはあり得なさそうだが...).
       _cache の値は GrowableCache::remove() 内で呼び出される GrowableCache::recache() によって 
       セットされるので, 呼び出し後には何もせずにリターン.)
      ---------------------------------------- -}

	  for (int i=0; i<len; i++) {
	    _cache[i] = _elements->at(i)->getCacheValue();
	    //
	    // The cache entry has gone bad. Without a valid frame pointer
	    // value, the entry is useless so we simply delete it in product
	    // mode. The call to remove() will rebuild the cache again
	    // without the bad entry.
	    //
	    if (_cache[i] == NULL) {
	      assert(false, "cannot recache NULL elements");
	      remove(i);
	      return;
	    }
	  }

  {- -------------------------------------------
  (1) _cache の最後には NULL を詰めておく.
      ---------------------------------------- -}

	  _cache[len] = NULL;
	
  {- -------------------------------------------
  (1) (_cache のアドレスを変更したので)
      _listener_fun フィールドに格納されているコールバックを呼び出しておく.
      ---------------------------------------- -}

	  _listener_fun(_this_obj,_cache);
	}
	
```


