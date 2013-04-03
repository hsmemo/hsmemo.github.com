---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateInterpreter_sparc.cpp
### 説明(description)

```
// Helper for vtos entry point generation

```

### 名前(function name)
```
void TemplateInterpreterGenerator::set_vtos_entry_points(Template* t, address& bep, address& cep, address& sep, address& aep, address& iep, address& lep, address& fep, address& dep, address& vep) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(t->is_valid() && t->tos_in() == vtos, "illegal template");

  {- -------------------------------------------
  (1) TemplateInterpreterGenerator::generate_and_dispatch() でコードを生成する.
  
      各エントリポイントについては, 以下のように設定する.
      * vtos で到達した場合のエントリ (= vep) については, 
        想定されている TOS 状態 (= vtos) とあっているため
        そのまま TemplateInterpreterGenerator::generate_and_dispatch() が生成したコードにフォールスルーするエントリを設定.
      * それ以外の TOS 状態で到達した場合のエントリについては, 
        想定されている TOS 状態 (= vtos) に合わせるために TOS の値を push してから 
        TemplateInterpreterGenerator::generate_and_dispatch() が生成したコードにフォールスルーするエントリを設定.
      ---------------------------------------- -}

	  Label L;
	  aep = __ pc(); __ push_ptr(); __ ba(false, L); __ delayed()->nop();
	  fep = __ pc(); __ push_f();   __ ba(false, L); __ delayed()->nop();
	  dep = __ pc(); __ push_d();   __ ba(false, L); __ delayed()->nop();
	  lep = __ pc(); __ push_l();   __ ba(false, L); __ delayed()->nop();
	  iep = __ pc(); __ push_i();
	  bep = cep = sep = iep;                        // there aren't any
	  vep = __ pc(); __ bind(L);                    // fall through
	  generate_and_dispatch(t);
	}
	
```


