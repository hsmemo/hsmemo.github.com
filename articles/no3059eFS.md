---
layout: default
title: Serviceability 機能 ： JVMTI 処理の概要 ： interp_only_mode  
---
[Up](no3718uqQ.html) [Top](../index.html)

#### Serviceability 機能 ： JVMTI 処理の概要 ： interp_only_mode  

--- 
## 概要(Summary)
一部の JVMTI イベントに関しては, イベント通知を有効にすると HotSpot の実行が interpreter だけに制限される ("interp_only_mode").
interp_only_mode の間は HotSpot の挙動が以下のように変わる.

  * メソッド呼び出しの際に, (from_interpreted_entry() ではなく) 常に interpreter_entry() が使われる.
  * CompilationPolicy も計測処理を行わなくなる.
  * (その他に, MethodEntry や MethodExit といった JVMTI イベントの通知部でも
    interp_only_mode でなければ通知処理は止めてリターン, という処理になっていたりする)

なお, 有効にすると interp_only_mode になるイベント種別は以下の通り
(基本的には半端なく重いイベント通知の場合にのみ interp_only_mode になる).

  * SingleStep (1命令毎に通知)
  * FieldAccess	 (フィールドへの読み書き毎に通知)
  * FieldModification (〃)
  * MethodEntry (関数のコール/リターン毎に通知)
  * MethodExit (〃)
  * FramePop (関数のリターン毎に通知)


```
    ((cite: hotspot/src/share/vm/prims/jvmtiEventController.cpp))
    static const jlong  INTERP_EVENT_BITS =  SINGLE_STEP_BIT | METHOD_ENTRY_BIT | METHOD_EXIT_BIT |
                                    FRAME_POP_BIT | FIELD_ACCESS_BIT | FIELD_MODIFICATION_BIT;
```


## 備考(Notes)
* interp_only_mode になっているかどうかは, JavaThread 内の _interp_only_mode フィールドで管理している.


```
    ((cite: hotspot/src/share/vm/runtime/thread.hpp))
      // Used by the interpreter in fullspeed mode for frame pop, method
      // entry, method exit and single stepping support. This field is
      // only set to non-zero by the VM_EnterInterpOnlyMode VM operation.
      // It can be set to zero asynchronously (i.e., without a VM operation
      // or a lock) so we have to be very careful.
      int               _interp_only_mode;
```

* interp_only_mode の on/off の切り替えは
  JvmtiEventControllerPrivate::enter_interp_only_mode() と JvmtiEventControllerPrivate::leave_interp_only_mode() で行っている.
  
  (コメントに書かれている通り,
  有効にする際には VM operation (VM_EnterInterpOnlyMode::doit()) で切り替えているが,
  無効にする際には単にフィールドを書き換えるだけでロックも取らない)
  
  (なお, 当然ながら有効にする際に deopt も行うので,
  当該スレッドでは一切 compiled code は実行されなくなる)

* なお, interp_only_mode になるイベントに付いては,
  イベントを有効にしなくても, 
  関連する capability を取得した時点で既にいくつかの最適化が切られた状態になる
  (See: JvmtiManageCapabilities::update()).


```
    ((cite: hotspot/src/share/vm/prims/jvmtiManageCapabilities.cpp))
        // Disable these when tracking the bytecodes
        UseFastEmptyMethods = false;
        UseFastAccessorMethods = false;
```

* 内部実装的には, 
  capability 処理の init_onload_capabilities() と init_onload_solo_capabilities() で 
  JVMTI 関係のビットが on にされ,
  その後に JvmtiManageCapabilities::update() 内で JvmtiExport::can_post_interpreter_events の値が変更されることで, JvmtiExport::can_post_interpreter_events() が true を返すようになる.
  
  そして, この値によって C++ interpreter では run() / runWithChecks() の切り替えが行われ,
  template interpreter では生成コードに JVMTI 関係のチェックを挿入するかどうかの切り替えが行われる模様.
  
  (なお, 一見すると JvmtiExport::can_post_interpreter_events の値が 1 になる処理パスがないように見えるが, onload は一度取得されると always に移されるので実際にはあり得る)

  (See: [here](no2935trw.html) for details)


## 備考(Notes)
interp_only_mode の有無により挙動が変わる箇所は以下の通り.

### メソッド呼び出しの際に, (from_interpreted_entry() ではなく) 常に interpreter_entry() が使われる
#### JavaCalls::call_helper()
(まず methodOopDesc::from_interpreted_offset() からロードするが,
interp_only_mode であれば methodOopDesc::interpreter_entry_offset() の値をロードしなおす)


```
    ((cite: hotspot/src/share/vm/runtime/javaCalls.cpp))
      // Since the call stub sets up like the interpreter we call the from_interpreted_entry
      // so we can go compiled via a i2c. Otherwise initial entry method will always
      // run interpreted.
      address entry_point = method->from_interpreted_entry();
      if (JvmtiExport::can_post_interpreter_events() && thread->is_interp_only_mode()) {
        entry_point = method->interpreter_entry();
      }
```

#### InterpreterMacroAssembler::call_from_interpreter() : (sparc の場合)
(まず methodOopDesc::from_interpreted_offset() からロードするが,
interp_only_mode であれば methodOopDesc::interpreter_entry_offset() の値をロードしなおす)


```
    ((cite: hotspot/src/cpu/sparc/vm/interp_masm_sparc.cpp))
      // Assume we want to go compiled if available
    
      ld_ptr(G5_method, in_bytes(methodOopDesc::from_interpreted_offset()), target);
    
      if (JvmtiExport::can_post_interpreter_events()) {
        // JVMTI events, such as single-stepping, are implemented partly by avoiding running
        // compiled code in threads for which the event is enabled.  Check here for
        // interp_only_mode if these events CAN be enabled.
        verify_thread();
        Label skip_compiled_code;
    
        const Address interp_only(G2_thread, JavaThread::interp_only_mode_offset());
        ld(interp_only, scratch);
        tst(scratch);
        br(Assembler::notZero, true, Assembler::pn, skip_compiled_code);
        delayed()->ld_ptr(G5_method, in_bytes(methodOopDesc::interpreter_entry_offset()), target);
        bind(skip_compiled_code);
      }
```

#### InterpreterMacroAssembler::jump_from_interpreted() : (x86_64 の場合)
(interp_only_mode であれば methodOopDesc::interpreter_entry_offset() にジャンプする.
そうでなければ, methodOopDesc::from_interpreted_offset() にジャンプする)


```
    ((cite: hotspot/src/cpu/x86/vm/interp_masm_x86_64.cpp))
      prepare_to_jump_from_interpreted();
    
      if (JvmtiExport::can_post_interpreter_events()) {
        Label run_compiled_code;
        // JVMTI events, such as single-stepping, are implemented partly by avoiding running
        // compiled code in threads for which the event is enabled.  Check here for
        // interp_only_mode if these events CAN be enabled.
        // interp_only is an int, on little endian it is sufficient to test the byte only
        // Is a cmpl faster?
        cmpb(Address(r15_thread, JavaThread::interp_only_mode_offset()), 0);
        jcc(Assembler::zero, run_compiled_code);
        jmp(Address(method, methodOopDesc::interpreter_entry_offset()));
        bind(run_compiled_code);
      }
    
      jmp(Address(method, methodOopDesc::from_interpreted_offset()));
```


### CompilationPolicy も計測処理(& CompilerBroker の呼び出し)を行わなくなる
#### NonTieredCompPolicy::event()

```
    ((cite: hotspot/src/share/vm/runtime/compilationPolicy.cpp))
      if (JvmtiExport::can_post_interpreter_events()) {
        assert(THREAD->is_Java_thread(), "Wrong type of thread");
        if (((JavaThread*)THREAD)->is_interp_only_mode()) {
          // If certain JVMTI events (e.g. frame pop event) are requested then the
          // thread is forced to remain in interpreted code. This is
          // implemented partly by a check in the run_compiled_code
          // section of the interpreter whether we should skip running
          // compiled code, and partly by skipping OSR compiles for
          // interpreted-only threads.
          if (bci != InvocationEntryBci) {
            reset_counter_for_back_branch_event(method);
            return NULL;
          }
        }
      }
```

#### SimpleThresholdPolicy::event()

```
    ((cite: hotspot/src/share/vm/runtime/simpleThresholdPolicy.cpp))
      if (comp_level == CompLevel_none &&
          JvmtiExport::can_post_interpreter_events()) {
        assert(THREAD->is_Java_thread(), "Should be java thread");
        if (((JavaThread*)THREAD)->is_interp_only_mode()) {
          return NULL;
        }
      }
```







