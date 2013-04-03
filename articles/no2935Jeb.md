---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.cpp

### 名前(function name)
```
void methodOopDesc::set_breakpoint(int bci) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  instanceKlass* ik = instanceKlass::cast(method_holder());

  {- -------------------------------------------
  (1) 新しい BreakpointInfo オブジェクトを生成し, 
      対応する instanceKlass 内の breakpoints リストにつなぐ.
      ---------------------------------------- -}

	  BreakpointInfo *bp = new BreakpointInfo(this, bci);
	  bp->set_next(ik->breakpoints());
	  ik->set_breakpoints(bp);

  {- -------------------------------------------
  (1) BreakpointInfo::set() を呼んで, 
      該当個所のバイトコードを breakpoint 命令に書き換える.
      ---------------------------------------- -}

	  // do this last:
	  bp->set(this);
	}
	
```


