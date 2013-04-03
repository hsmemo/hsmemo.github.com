---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/virtualspace.cpp

### 名前(function name)
```
void ReservedSpace::protect_noaccess_prefix(const size_t size) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert( (_noaccess_prefix != 0) == (UseCompressedOops && _base != NULL &&
	                                      (size_t(_base + _size) > OopEncodingHeapMax) &&
	                                      Universe::narrow_oop_use_implicit_null_checks()),
	         "noaccess_prefix should be used only with non zero based compressed oops");
	
  {- -------------------------------------------
  (1) もし access prefix が不要であれば, ここでリターン
      ---------------------------------------- -}

	  // If there is no noaccess prefix, return.
	  if (_noaccess_prefix == 0) return;
	
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(_noaccess_prefix >= (size_t)os::vm_page_size(),
	         "must be at least page size big");
	
  {- -------------------------------------------
  (1) os::protect_memory() で, ヒープの先頭を access prefix 分だけアクセス禁止にする.
      失敗したら, fatal() によりここで異常終了.
      ---------------------------------------- -}

	  // Protect memory at the base of the allocated region.
	  // If special, the page was committed (only matters on windows)
	  if (!os::protect_memory(_base, _noaccess_prefix, os::MEM_PROT_NONE,
	                          _special)) {
	    fatal("cannot protect protection page");
	  }

  {- -------------------------------------------
  (1) (トレース出力)
      ---------------------------------------- -}

	  if (PrintCompressedOopsMode) {
	    tty->cr();
	    tty->print_cr("Protected page at the reserved heap base: " PTR_FORMAT " / " INTX_FORMAT " bytes", _base, _noaccess_prefix);
	  }
	
  {- -------------------------------------------
  (1) アクセス禁止にした分だけ _base と _size を微調整しておく
      ---------------------------------------- -}

	  _base += _noaccess_prefix;
	  _size -= _noaccess_prefix;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert((size == _size) && ((uintptr_t)_base % _alignment == 0),
	         "must be exactly of required size and alignment");
	}
	
```


