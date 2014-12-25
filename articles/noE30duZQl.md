---
layout: default
title: CompilerOracle クラス (CompilerOracle, 及びその補助クラス(MethodMatcher, MethodOptionMatcher))
---
[Top](../index.html)

#### CompilerOracle クラス (CompilerOracle, 及びその補助クラス(MethodMatcher, MethodOptionMatcher))

これらは, ユーザーが JIT コンパイラを制御するためのクラス (See: [here](no7882MiN.html) and [here](noh7R3dY_J.html) for details).


### クラス一覧(class list)

  * [CompilerOracle](#noJlkFgwZo)
  * [MethodMatcher](#noQD7ffwLM)
  * [MethodOptionMatcher](#noBJd4ieRE)


---
## <a name="noJlkFgwZo" id="noJlkFgwZo">CompilerOracle</a>

### 概要(Summary)
JIT コンパイラに対するユーザーからの指示を扱うためのクラス
(より正確には, そのための機能を納めた名前空間(AllStatic クラス)).

以下のような指示を出せる模様.

* コンパイル対象としないメソッドの指定
* インライン展開して欲しいメソッド/して欲しくないメソッドの指定
* 生成後のアセンブリ列を表示して欲しいメソッドの指定
* JIT コンパイルを行ったことをログに残して欲しいメソッドの指定
* JIT 生成コードの先頭にブレークポイントを埋め込んで欲しいメソッドの指定
* etc

なお, 指示は CompileCommandFile オプションで指定したファイルに記述する
(指定しなかった場合は ".hotspot_compiler" というファイルが使われる).


```cpp
    ((cite: hotspot/src/share/vm/compiler/compilerOracle.hpp))
    // CompilerOracle is an interface for turning on and off compilation
    // for some methods
    
    class CompilerOracle : AllStatic {
```

### 備考(Notes)
以下のようなコマンドに対応している模様.


```cpp
    ((cite: hotspot/src/share/vm/compiler/compilerOracle.cpp))
    // this must parallel the enum OracleCommand
    static const char * command_names[] = {
      "break",
      "print",
      "exclude",
      "inline",
      "dontinline",
      "compileonly",
      "log",
      "option",
      "quiet",
      "help"
    };
```

また, ".hotspot_compiler" に help とだけ書いて java を実行させると以下の usage が表示されるので, それも参考のこと.


```cpp
    ((cite: hotspot/src/share/vm/compiler/compilerOracle.cpp))
    static void usage() {
      tty->print_cr("  CompileCommand and the CompilerOracle allows simple control over");
      tty->print_cr("  what's allowed to be compiled.  The standard supported directives");
      tty->print_cr("  are exclude and compileonly.  The exclude directive stops a method");
      tty->print_cr("  from being compiled and compileonly excludes all methods except for");
      tty->print_cr("  the ones mentioned by compileonly directives.  The basic form of");
      tty->print_cr("  all commands is a command name followed by the name of the method");
      tty->print_cr("  in one of two forms: the standard class file format as in");
      tty->print_cr("  class/name.methodName or the PrintCompilation format");
      tty->print_cr("  class.name::methodName.  The method name can optionally be followed");
      tty->print_cr("  by a space then the signature of the method in the class file");
      tty->print_cr("  format.  Otherwise the directive applies to all methods with the");
      tty->print_cr("  same name and class regardless of signature.  Leading and trailing");
      tty->print_cr("  *'s in the class and/or method name allows a small amount of");
      tty->print_cr("  wildcarding.  ");
      tty->cr();
      tty->print_cr("  Examples:");
      tty->cr();
      tty->print_cr("  exclude java/lang/StringBuffer.append");
      tty->print_cr("  compileonly java/lang/StringBuffer.toString ()Ljava/lang/String;");
      tty->print_cr("  exclude java/lang/String*.*");
      tty->print_cr("  exclude *.toString");
    }
```




### 詳細(Details)
See: [here](../doxygen/classCompilerOracle.html) for details

---
## <a name="noQD7ffwLM" id="noQD7ffwLM">MethodMatcher</a>

### 概要(Summary)
CompilerOracle クラスの補助クラス.

(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/compiler/compilerOracle.cpp))
    class MethodMatcher : public CHeapObj {
```



### 詳細(Details)
See: [here](../doxygen/classMethodMatcher.html) for details

---
## <a name="noBJd4ieRE" id="noBJd4ieRE">MethodOptionMatcher</a>

### 概要(Summary)
CompilerOracle クラスの補助クラス.

(#Under Construction)


```cpp
    ((cite: hotspot/src/share/vm/compiler/compilerOracle.cpp))
    class MethodOptionMatcher: public MethodMatcher {
```




### 詳細(Details)
See: [here](../doxygen/classMethodOptionMatcher.html) for details

---
