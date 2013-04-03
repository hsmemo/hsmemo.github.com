---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/sparc/vm/templateTable_sparc.cpp

### 名前(function name)
```
void TemplateTable::irem() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (assert) (See: TemplateTable::transition())
      ---------------------------------------- -}

	  transition(itos, itos);

  {- -------------------------------------------
  (1) コード生成:
      「divosor を O2 レジスタに待避しておく」
      ---------------------------------------- -}

	  __ mov(Otos_i, O2); // save divisor

  {- -------------------------------------------
  (1) コード生成:
      「X mod Y = X - (X/Y)*Y  なので, その通りに計算する.」
  
      (なおコメントによると, このコードは idiv() が 
       dividend を O1 レジスタに残すことを想定している, とのこと)
      ---------------------------------------- -}

	  idiv();                               // %%%% Hack: exploits fact that idiv leaves dividend in O1
	  __ smul(Otos_i, O2, Otos_i);
	  __ sub(O1, Otos_i, Otos_i);
	}
	
```


