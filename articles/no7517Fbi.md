---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/constantPoolOop.cpp

### 名前(function name)
```
klassOop constantPoolOopDesc::klass_at_impl(constantPoolHandle this_oop, int which, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) まず, Constant Pool 内の対象箇所が resolve 済みかどうかをチェックする.
      resolve 済みであれば, それをリターンするだけ.
  
      (なおコメントによると, ここでは tag bit で判定するのは危険, とのこと)
      ---------------------------------------- -}

	  // A resolved constantPool entry will contain a klassOop, otherwise a Symbol*.
	  // It is not safe to rely on the tag bit's here, since we don't have a lock, and the entry and
	  // tag is not updated atomicly.
	  CPSlot entry = this_oop->slot_at(which);
	  if (entry.is_oop()) {
	    assert(entry.get_oop()->is_klass(), "must be");
	    // Already resolved - return entry.
	    return (klassOop)entry.get_oop();
	  }
	
  {- -------------------------------------------
  (1) (以下のブロック内の処理は ... で排他した状態で行う)
      Constant Pool 内の対象箇所が resolve 済みかどうかを (ロックを取った状態でもう一度) チェックする.
      結果に応じて以下の局所変数を設定.
  
      * in_error
        以前に実行された解決処理でエラーが起こっていた場合は, true
  
      * do_resolve
        まだ解決されておらず, 解決処理でエラーも出てなければ, true
      * name
        まだ解決されておらず, 解決処理でエラーも出てなければ, 解決対象のクラス名
      * loader
        まだ解決されておらず, 解決処理でエラーも出てなければ, 使用するクラスローダー
        (「この constantPoolOopDesc が属するクラスオブジェクト」をロードしたクラスローダーを使用)
      ---------------------------------------- -}

	  // Acquire lock on constant oop while doing update. After we get the lock, we check if another object
	  // already has updated the object
	  assert(THREAD->is_Java_thread(), "must be a Java thread");
	  bool do_resolve = false;
	  bool in_error = false;
	
	  Symbol* name = NULL;
	  Handle       loader;
	  { ObjectLocker ol(this_oop, THREAD);
	
	    if (this_oop->tag_at(which).is_unresolved_klass()) {
	      if (this_oop->tag_at(which).is_unresolved_klass_in_error()) {
	        in_error = true;
	      } else {
	        do_resolve = true;
	        name   = this_oop->unresolved_klass_at(which);
	        loader = Handle(THREAD, instanceKlass::cast(this_oop->pool_holder())->class_loader());
	      }
	    }
	  } // unlocking constantPool
	
	
  {- -------------------------------------------
  (1) 以前に実行された解決処理でエラーが起こっていた場合は, ここで終了
      (SystemDictionary::find_resolution_error() で以前に発生した例外名を取得し, それを投げるだけ)
  
      (JVMS 5.4.3 の要請に従い, 一度起こった例外は記録し, 次回以降は同じ例外が発生するようにしている)
      ---------------------------------------- -}

	  // The original attempt to resolve this constant pool entry failed so find the
	  // original error and throw it again (JVMS 5.4.3).
	  if (in_error) {
	    Symbol* error = SystemDictionary::find_resolution_error(this_oop, which);
	    guarantee(error != (Symbol*)NULL, "tag mismatch with resolution error table");
	    ResourceMark rm;
	    // exception text will be the class name
	    const char* className = this_oop->unresolved_klass_at(which)->as_C_string();
	    THROW_MSG_0(error, className);
	  }
	
  {- -------------------------------------------
  (1) まだ解決されてなければ, 以下の if ブロック内で解決処理を行う.
      ---------------------------------------- -}

	  if (do_resolve) {

    {- -------------------------------------------
  (1.1) SystemDictionary::resolve_or_fail() を呼んで, 
        対象のクラス／インターフェースを取得する.
        ---------------------------------------- -}

	    // this_oop must be unlocked during resolve_or_fail
	    oop protection_domain = Klass::cast(this_oop->pool_holder())->protection_domain();
	    Handle h_prot (THREAD, protection_domain);
	    klassOop k_oop = SystemDictionary::resolve_or_fail(name, loader, h_prot, true, THREAD);

    {- -------------------------------------------
  (1.1) 
        ---------------------------------------- -}

	    KlassHandle k;
	    if (!HAS_PENDING_EXCEPTION) {
	      k = KlassHandle(THREAD, k_oop);
	      // Do access check for klasses
	      verify_constant_pool_resolve(this_oop, k, THREAD);
	    }
	
    {- -------------------------------------------
  (1.1) もし解決処理で例外が出ていた場合は, 
        以下のどれかの値をリターンする、あるいはどれかの例外を送出する.
        (JVMS 5.4.3 の要請に従い, 一度起こった例外は記録し, 次回以降は同じ例外が発生するようにしている)
  
        * 既に Constant Pool 内の対象箇所が解決済みだった場合: (= 他のスレッドが解決した場合)
          (解決済みならそれを使えばいいだけなので) その内容をリターン
  
        * 出ていた例外が LinkageError ではない場合:
          単に 0 をリターンするだけ (発生した例外がそのまま持ち上がる).
          (コメントによると, 
  	JVMS 5.4.3 の要請では一度エラーになったら次回以降は同じ例外を出さないといけないが, 
          virtual machine errors (StackOverflow, OutOfMemoryError, etc) については
  	次回以降も出すのは意図するところと違うので, 
          今回は (記録はせず) 単に例外をそのまま持ち上げるだけにしておく, とのこと.
  	(詳細は Bug ID 6308271))
          
        * 出ていた例外は LinkageError だが, まだエラーが記録されていない場合:
  	SystemDictionary::add_resolution_error() を呼んで, 発生した例外を記録しておく.
  	また, constantPoolOop::tag_at_put() を呼んで, 解決が失敗したことを記録しておく.
  	最後に, そのまま 0 をリターン (発生した LinkageError がそのまま持ち上がる).
  
        * 出ていた例外が LinkageError であり, 既にエラーが記録されている場合
          SystemDictionary::find_resolution_error() で以前に発生した例外名を取得し, それを投げる.
        ---------------------------------------- -}

	    // Failed to resolve class. We must record the errors so that subsequent attempts
	    // to resolve this constant pool entry fail with the same error (JVMS 5.4.3).
	    if (HAS_PENDING_EXCEPTION) {
	      ResourceMark rm;
	      Symbol* error = PENDING_EXCEPTION->klass()->klass_part()->name();
	
	      bool throw_orig_error = false;
	      {
	        ObjectLocker ol (this_oop, THREAD);
	
	        // some other thread has beaten us and has resolved the class.
	        if (this_oop->tag_at(which).is_klass()) {
	          CLEAR_PENDING_EXCEPTION;
	          entry = this_oop->resolved_klass_at(which);
	          return (klassOop)entry.get_oop();
	        }
	
	        if (!PENDING_EXCEPTION->
	              is_a(SystemDictionary::LinkageError_klass())) {
	          // Just throw the exception and don't prevent these classes from
	          // being loaded due to virtual machine errors like StackOverflow
	          // and OutOfMemoryError, etc, or if the thread was hit by stop()
	          // Needs clarification to section 5.4.3 of the VM spec (see 6308271)
	        }
	        else if (!this_oop->tag_at(which).is_unresolved_klass_in_error()) {
	          SystemDictionary::add_resolution_error(this_oop, which, error);
	          this_oop->tag_at_put(which, JVM_CONSTANT_UnresolvedClassInError);
	        } else {
	          // some other thread has put the class in error state.
	          error = SystemDictionary::find_resolution_error(this_oop, which);
	          assert(error != NULL, "checking");
	          throw_orig_error = true;
	        }
	      } // unlocked
	
	      if (throw_orig_error) {
	        CLEAR_PENDING_EXCEPTION;
	        ResourceMark rm;
	        const char* className = this_oop->unresolved_klass_at(which)->as_C_string();
	        THROW_MSG_0(error, className);
	      }
	
	      return 0;
	    }
	
    {- -------------------------------------------
  (1.1) TraceClassResolution が指定されている場合, (かつ対象が配列クラスではない場合)
        クラスが参照される度にトレースを出したいので
        あえて Constant Pool を更新しないままにしておく.
  
        (トレース出力) を出し, そのまま結果をリターンするだけ.
        ---------------------------------------- -}

	    if (TraceClassResolution && !k()->klass_part()->oop_is_array()) {
	      // skip resolving the constant pool so that this code get's
	      // called the next time some bytecodes refer to this class.
	      ResourceMark rm;
	      int line_number = -1;
	      const char * source_file = NULL;
	      if (JavaThread::current()->has_last_Java_frame()) {
	        // try to identify the method which called this function.
	        vframeStream vfst(JavaThread::current());
	        if (!vfst.at_end()) {
	          line_number = vfst.method()->line_number_from_bci(vfst.bci());
	          Symbol* s = instanceKlass::cast(vfst.method()->method_holder())->source_file_name();
	          if (s != NULL) {
	            source_file = s->as_C_string();
	          }
	        }
	      }
	      if (k() != this_oop->pool_holder()) {
	        // only print something if the classes are different
	        if (source_file != NULL) {
	          tty->print("RESOLVE %s %s %s:%d\n",
	                     instanceKlass::cast(this_oop->pool_holder())->external_name(),
	                     instanceKlass::cast(k())->external_name(), source_file, line_number);
	        } else {
	          tty->print("RESOLVE %s %s\n",
	                     instanceKlass::cast(this_oop->pool_holder())->external_name(),
	                     instanceKlass::cast(k())->external_name());
	        }
	      }
	      return k();

    {- -------------------------------------------
  (1.1) トレース出力を出さない場合は, 
        constantPoolOopDesc::klass_at_put() を呼んで Constant Pool の該当箇所を更新する.
        (次回以降に解決の必要性をなくすため. ただし, 既に解決済みの場合は更新しない)
  
        (なお, この処理は ... を取得して排他した状態で行う)
        ---------------------------------------- -}

	    } else {
	      ObjectLocker ol (this_oop, THREAD);
	      // Only updated constant pool - if it is resolved.
	      do_resolve = this_oop->tag_at(which).is_unresolved_klass();
	      if (do_resolve) {
	        this_oop->klass_at_put(which, k());
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	  entry = this_oop->resolved_klass_at(which);
	  assert(entry.is_oop() && entry.get_oop()->is_klass(), "must be resolved at this point");
	  return (klassOop)entry.get_oop();
	}
	
```


