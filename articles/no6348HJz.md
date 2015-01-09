---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/runtime/compilationPolicy.cpp

### 名前(function name)
```
bool CompilationPolicy::is_compilation_enabled() {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 以下の条件が両方とも成立していれば true をリターンする. そうでなければ false をリターンする.
      * 既に起動処理が終わっている (= CompilationPolicy::delay_compilation_during_startup() が false)
      * JIT コンパイラが次の作業を開始してよい状況である (= CompileBroker::should_compile_new_jobs() が true)
      ---------------------------------------- -}

	  // NOTE: CompileBroker::should_compile_new_jobs() checks for UseCompiler
	  return !delay_compilation_during_startup() && CompileBroker::should_compile_new_jobs();
	}
	
```


