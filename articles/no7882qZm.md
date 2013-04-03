---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/interpreter/templateInterpreter.cpp

### 名前(function name)
```
void TemplateInterpreter::notice_safepoints() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (既に変更済みであれば, 何もしない)
      ---------------------------------------- -}

	  if (!_notice_safepoints) {

  {- -------------------------------------------
  (1) copy_table() を呼んで, _safept_table の内容を _active_table にコピーする.
  
      (なお, _safept_table の中身は Template Interpreter の構築時に作成されている.
       See: TemplateInterpreterGenerator::set_safepoints_for_all_bytes())
      ---------------------------------------- -}

	    // switch to safepoint dispatch table
	    _notice_safepoints = true;
	    copy_table((address*)&_safept_table, (address*)&_active_table, sizeof(_active_table) / sizeof(address));
	  }
	}
	
```


