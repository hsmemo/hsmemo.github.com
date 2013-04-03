---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/cpu/x86/vm/templateInterpreter_x86_64.cpp
### 説明(description)

```
// Entry points
//
// Here we generate the various kind of entries into the interpreter.
// The two main entry type are generic bytecode methods and native
// call method.  These both come in synchronized and non-synchronized
// versions but the frame layout they create is very similar. The
// other method entry types are really just special purpose entries
// that are really entry and interpretation all in one. These are for
// trivial methods like accessor, empty, or special math methods.
//
// When control flow reaches any of the entry types for the interpreter
// the following holds ->
//
// Arguments:
//
// rbx: methodOop
//
// Stack layout immediately at entry
//
// [ return address     ] <--- rsp
// [ parameter n        ]
//   ...
// [ parameter 1        ]
// [ expression stack   ] (caller's java expression stack)

// Assuming that we don't go to one of the trivial specialized entries
// the stack will look like below when we are ready to execute the
// first bytecode (or call the native routine). The register usage
// will be as the template based interpreter expects (see
// interpreter_amd64.hpp).
//
// local variables follow incoming parameters immediately; i.e.
// the return address is moved to the end of the locals).
//
// [ monitor entry      ] <--- rsp
//   ...
// [ monitor entry      ]
// [ expr. stack bottom ]
// [ saved r13          ]
// [ current r14        ]
// [ methodOop          ]
// [ saved ebp          ] <--- rbp
// [ return address     ]
// [ local variable m   ]
//   ...
// [ local variable 1   ]
// [ parameter n        ]
//   ...
// [ parameter 1        ] <--- r14

```

### 名前(function name)
```
address AbstractInterpreterGenerator::generate_method_entry(
                                        AbstractInterpreter::MethodKind kind) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // determine code generation flags
	  bool synchronized = false;
	  address entry_point = NULL;
	
  {- -------------------------------------------
  (1) 以下でメソッドエントリ部の処理を行うコードを生成し, その先頭アドレスをリターンする.
  
      引数で指定されたメソッド種別(kind)に応じて, 以下の関数を呼び分けている.
      (InterpreterGenerator::generate_normal_entry() は, 他のどれにも当てはまらない一般的な場合に使用)
      * InterpreterGenerator::generate_abstract_entry()     
      * InterpreterGenerator::generate_accessor_entry()     
      * InterpreterGenerator::generate_empty_entry()        
      * InterpreterGenerator::generate_math_entry()         
      * InterpreterGenerator::generate_method_handle_entry()
      * InterpreterGenerator::generate_Reference_get_entry()
      * InterpreterGenerator::generate_native_entry()
      * InterpreterGenerator::generate_normal_entry()
    
      (なお, InterpreterGenerator::generate_native_entry() と 
       InterpreterGenerator::generate_normal_entry() のケースについては, 
       synchronized かどうかに応じて呼び出し時の引数の true/false が変わる.)
      ---------------------------------------- -}

	  switch (kind) {
	  case Interpreter::zerolocals             :                                                                             break;
	  case Interpreter::zerolocals_synchronized: synchronized = true;                                                        break;
	  case Interpreter::native                 : entry_point = ((InterpreterGenerator*) this)->generate_native_entry(false); break;
	  case Interpreter::native_synchronized    : entry_point = ((InterpreterGenerator*) this)->generate_native_entry(true);  break;
	  case Interpreter::empty                  : entry_point = ((InterpreterGenerator*) this)->generate_empty_entry();       break;
	  case Interpreter::accessor               : entry_point = ((InterpreterGenerator*) this)->generate_accessor_entry();    break;
	  case Interpreter::abstract               : entry_point = ((InterpreterGenerator*) this)->generate_abstract_entry();    break;
	  case Interpreter::method_handle          : entry_point = ((InterpreterGenerator*) this)->generate_method_handle_entry();break;
	
	  case Interpreter::java_lang_math_sin     : // fall thru
	  case Interpreter::java_lang_math_cos     : // fall thru
	  case Interpreter::java_lang_math_tan     : // fall thru
	  case Interpreter::java_lang_math_abs     : // fall thru
	  case Interpreter::java_lang_math_log     : // fall thru
	  case Interpreter::java_lang_math_log10   : // fall thru
	  case Interpreter::java_lang_math_sqrt    : entry_point = ((InterpreterGenerator*) this)->generate_math_entry(kind);    break;
	  case Interpreter::java_lang_ref_reference_get
	                                           : entry_point = ((InterpreterGenerator*)this)->generate_Reference_get_entry(); break;
	  default                                  : ShouldNotReachHere();                                                       break;
	  }
	
	  if (entry_point) {
	    return entry_point;
	  }
	
	  return ((InterpreterGenerator*) this)->
	                                generate_normal_entry(synchronized);
	}
	
```


