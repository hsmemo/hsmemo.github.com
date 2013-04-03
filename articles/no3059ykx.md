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
void TemplateInterpreter::initialize() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 既に初期化が終わっていれば, することはないので, ここでリターン
      (_code は TemplateInterpreter::initialize() でしか設定しないように見えるが, 
       TemplateInterpreter::initialize() が複数回呼ばれることがある?? #TODO)
      ---------------------------------------- -}

	  if (_code != NULL) return;

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  // assertions
	  assert((int)Bytecodes::number_of_codes <= (int)DispatchTable::length,
	         "dispatch table too small");
	
  {- -------------------------------------------
  (1) AbstractInterpreter::initialize() を呼び出して, InvocationCounter 等の初期化を行う.
      ---------------------------------------- -}

	  AbstractInterpreter::initialize();
	
  {- -------------------------------------------
  (1) TemplateTable::initialize() を呼び出して, 各バイトコードに対応する Template オブジェクトを生成.
      (なお, ここでは Template オブジェクトを作るだけ. 
       これらの Template オブジェクトによって実際にコードが生成されるのはもう少し先)
      ---------------------------------------- -}

	  TemplateTable::initialize();
	
  {- -------------------------------------------
  (1) (以下のブロック内で Template Interpreter を構築する.
       実際の構築処理は InterpreterGenerator のコンストラクタ内で行われる.)
      ---------------------------------------- -}

	  // generate interpreter
	  { ResourceMark rm;

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    TraceTime timer("Interpreter generation", TraceStartupTime);

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    int code_size = InterpreterCodeSize;
	    NOT_PRODUCT(code_size *= 4;)  // debug uses extra interpreter code space

    {- -------------------------------------------
  (1.1) これから作成する Template コードを入れるための StubQueue を生成.
        (See: StubQueue)
        ---------------------------------------- -}

	    _code = new StubQueue(new InterpreterCodeletInterface, code_size, NULL,
	                          "Interpreter");

    {- -------------------------------------------
  (1.1) InterpreterGenerator のコンストラクタを呼び出して, Template Interpreter を構築する.
        ---------------------------------------- -}

	    InterpreterGenerator g(_code);

    {- -------------------------------------------
  (1.1) (トレース出力)
        ---------------------------------------- -}

	    if (PrintInterpreter) print();
	  }

  {- -------------------------------------------
  (1) (ここまでが Template Interpreter の構築処理)
      ---------------------------------------- -}
	
  {- -------------------------------------------
  (1) 生成したテーブル(_normal_table) を dispatch table にセットする.
      ---------------------------------------- -}

	  // initialize dispatch table
	  _active_table = _normal_table;
	}
	
```


