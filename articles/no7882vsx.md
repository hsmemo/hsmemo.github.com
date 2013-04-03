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
void TemplateInterpreterGenerator::set_short_entry_points(Template* t, address& bep, address& cep, address& sep, address& aep, address& iep, address& lep, address& fep, address& dep, address& vep) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(t->is_valid(), "template must exist");

  {- -------------------------------------------
  (1) バイトコードに対応するコードを生成し, また各 TOS 状態用のエントリポイントアドレスを生成する.
      
      なお, 生成するテンプレートが想定している TOS 状態 (= Template::tos_in()) に応じて, 処理を少し変えている.
      * 想定する TOS 状態が vtos の場合: 
        TemplateInterpreterGenerator::set_vtos_entry_points() でコード, およびエントリポイントを生成する.
      * それ以外の場合: 
        TemplateInterpreterGenerator::generate_and_dispatch() でコードを生成する.
        各エントリポイントについては, 以下のように設定する.
        * vtos で到達した場合のエントリ (= vep) には, 想定している TOS 状態に合うように値を pop() してから
          TemplateInterpreterGenerator::generate_and_dispatch() が生成したコードにフォールスルーするエントリを設定.
        * 想定している TOS 状態で到達した場合のエントリには, 
          直接 TemplateInterpreterGenerator::generate_and_dispatch() が生成したコードのアドレスを設定.
        * それ以外の TOS 状態のエントリは, 変更しない 
          (初期値のままとし, 実行時に使用された場合にはエラーを起こすようにしておく)
          (See: TemplateInterpreterGenerator::set_entry_points())
      ---------------------------------------- -}

	  switch (t->tos_in()) {
	    case btos:
	    case ctos:
	    case stos:
	      ShouldNotReachHere();  // btos/ctos/stos should use itos.
	      break;
	    case atos: vep = __ pc(); __ pop(atos); aep = __ pc(); generate_and_dispatch(t); break;
	    case itos: vep = __ pc(); __ pop(itos); iep = __ pc(); generate_and_dispatch(t); break;
	    case ltos: vep = __ pc(); __ pop(ltos); lep = __ pc(); generate_and_dispatch(t); break;
	    case ftos: vep = __ pc(); __ pop(ftos); fep = __ pc(); generate_and_dispatch(t); break;
	    case dtos: vep = __ pc(); __ pop(dtos); dep = __ pc(); generate_and_dispatch(t); break;
	    case vtos: set_vtos_entry_points(t, bep, cep, sep, aep, iep, lep, fep, dep, vep);     break;
	    default  : ShouldNotReachHere();                                                 break;
	  }
	}
	
```


