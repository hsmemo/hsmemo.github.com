---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/memory/iterator.cpp

### 名前(function name)
```
void CodeBlobToOopClosure::do_code_blob(CodeBlob* cb) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) do_marking コンストラクタ引数の値に応じて, 以下のどちらかを行う.
      * do_marking が false の場合:
        nmethod::oops_do() を呼んで, コンストラクタ引数で受け取った OopClosure を適用するだけ
        (ただし, 処理対象の nmethod が NULL の場合には何も行わない (行わないというか, することがないというか...))
      * do_marking が true の場合: 
        MarkingCodeBlobClosure::do_code_blob() を呼んで, marking 処理を用いた呼び出しを行う
      ---------------------------------------- -}

	  if (!_do_marking) {
	    nmethod* nm = cb->as_nmethod_or_null();
	    NOT_PRODUCT(if (TraceScavenge && Verbose && nm != NULL)  nm->print_on(tty, "oops_do, unmarked visit\n"));
	    // This assert won't work, since there are lots of mini-passes
	    // (mostly in debug mode) that co-exist with marking phases.
	    //assert(!(cb->is_nmethod() && ((nmethod*)cb)->test_oops_do_mark()), "found marked nmethod during mark-free phase");
	    if (nm != NULL) {
	      nm->oops_do(_cl);
	    }
	  } else {
	    MarkingCodeBlobClosure::do_code_blob(cb);
	  }
	}
	
```


