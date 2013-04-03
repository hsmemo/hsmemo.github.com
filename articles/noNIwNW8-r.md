---
layout: default
title: Interpreter クラス関連のクラス (InterpreterCodelet, CodeletMark, Interpreter)
---
[Top](../index.html)

#### Interpreter クラス関連のクラス (InterpreterCodelet, CodeletMark, Interpreter)

これらは, Interpreter クラス (インタープリタの実行処理を司るクラス) 関連のプラットフォーム非依存な部分を定義するクラス (See: [here](no7882AgC.html) for details).

(なお, コメントでは interpreter generator の名前も挙がっているが, 
 それは hotspot/src/share/vm/interpreter/interpreterGenerator.hpp の方で定義されているような... #TODO)


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
    // This file contains the platform-independent parts
    // of the interpreter and the interpreter generator.
```



### クラス一覧(class list)

  * [Interpreter](#no81gUMEjX)
  * [InterpreterCodelet](#noNuDQ-OG1)
  * [CodeletMark](#noIfMQoEor)


---
## <a name="no81gUMEjX" id="no81gUMEjX">Interpreter</a>

### 概要(Summary)
AbstractInterpreter クラスの具象サブクラス的なクラス
(AllStatic クラスなのでそもそも abstract class でもないが役割としては具象サブクラスに近い). 
ビルド時の #ifdef によって TemplateInterpreter か CppInterpreter のどちらかのサブクラスになる.

実質的な役割としては, 使用する Interpreter 種別を隠蔽するためのラッパークラス.

実際の Interpreter クラスとしての機能は TemplateInterpreter クラスもしくは CppInterpreter クラスに定義されているが, 
実際の使用箇所では Interpreter 種別の違いを隠蔽したコードにするため
Interpreter クラスとしてアクセスされることが多い模様 (See: [here](no7882AgC.html) for details).


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
    // Wrapper classes to produce Interpreter/InterpreterGenerator from either
    // the c++ interpreter or the template interpreter.
    
    class Interpreter: public CC_INTERP_ONLY(CppInterpreter) NOT_CC_INTERP(TemplateInterpreter) {
```




### 詳細(Details)
See: [here](../doxygen/classInterpreter.html) for details

---
## <a name="noNuDQ-OG1" id="noNuDQ-OG1">InterpreterCodelet</a>

### 概要(Summary)
Stub クラスの具象サブクラスの1つ.

このクラスは, Interpreter クラスが使用するマシン語コード片 ("codelet") 用.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // An InterpreterCodelet is a piece of interpreter code. All
    // interpreter code is generated into little codelets which
    // contain extra information for debugging and printing purposes.
    
    class InterpreterCodelet: public Stub {
```

### 使われ方(Usage)
#### インスタンスの格納場所(where its instances are stored)
AbstractInterpreter クラスの _code フィールド (static フィールド) に(のみ)格納されている.

(正確には, このフィールドは StubQueue を格納するフィールド.
この中に生成された全ての InterpreterCodelet オブジェクトが格納されている)

#### 生成箇所(where its instances are created)
StubQueue::request() というファクトリメソッドが用意されており, その中で(のみ)生成されている.




### 詳細(Details)
See: [here](../doxygen/classInterpreterCodelet.html) for details

---
## <a name="noIfMQoEor" id="noIfMQoEor">CodeletMark</a>

### 概要(Summary)
AbstractInterpreterGenerator クラス(及びそのサブクラス)内で使用される補助クラス.

InterpreterCodelet オブジェクトの構築処理で使用される補助クラス.
構築処理の前準備や後片付けを行う.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
    //------------------------------------------------------------------------------------------------------------------------
    // A CodeletMark serves as an automatic creator/initializer for Codelets
    // (As a subclass of ResourceMark it automatically GC's the allocated
    // code buffer and assemblers).
    
    class CodeletMark: ResourceMark {
```

### 使われ方(Usage)
#### 使用方法の概要(how to use)
コード生成を行う箇所で, 局所変数として生成される. デストラクタで実際のコード生成処理が行われる.

#### 使用箇所(where its instances are used)
以下の箇所で(のみ)使用されている.

* AbstractInterpreterGenerator::generate_all()
* CppInterpreterGenerator::generate_all()
* TemplateInterpreterGenerator::generate_all()
* TemplateInterpreterGenerator::set_entry_points()

### 内部構造(Internal structure)
コンストラクタ内で AbstractInterpreter::_code フィールドの StubQueue 上に Stub 用の領域を確保している.
また, そこに書き込みを行うための CodeBuffer と InterpreterMacroAssembler も作成している.

(なお, 作成した InterpreterMacroAssembler は引数として渡された masm 参照に代入されて返される.
 実際に使うときには, Interpreter クラスの _masm フィールドを引数として渡すことが多い (というかそれ以外の使い方をされていない).
 InterpreterGenerator 内ではこの InterpreterMacroAssembler オブジェクトを用いてコード生成している)


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
      CodeletMark(
        InterpreterMacroAssembler*& masm,
        const char* description,
        Bytecodes::Code bytecode = Bytecodes::_illegal):
        _clet((InterpreterCodelet*)AbstractInterpreter::code()->request(codelet_size())),
        _cb(_clet->code_begin(), _clet->code_size())
    
      { // request all space (add some slack for Codelet data)
        assert (_clet != NULL, "we checked not enough space already");
    
        // initialize Codelet attributes
        _clet->initialize(description, bytecode);
        // create assembler for code generation
        masm  = new InterpreterMacroAssembler(&_cb);
        _masm = &masm;
      }
```

そして, デストラクタ内で AbstractInterpreter::_code フィールドの StubQueue に commit() している.


```
    ((cite: hotspot/src/share/vm/interpreter/interpreter.hpp))
      ~CodeletMark() {
        // align so printing shows nop's instead of random code at the end (Codelets are aligned)
        (*_masm)->align(wordSize);
        // make sure all code is in code buffer
        (*_masm)->flush();
    
    
        // commit Codelet
        AbstractInterpreter::code()->commit((*_masm)->code()->pure_insts_size());
        // make sure nobody can use _masm outside a CodeletMark lifespan
        *_masm = NULL;
      }
```




### 詳細(Details)
See: [here](../doxygen/classCodeletMark.html) for details

---
