---
layout: default
title: 実行時コンスタントプールのシンボルの解決処理(resolution) (2) ： 解決処理
---
[Up](no7882NqI.html) [Top](../index.html)

#### 実行時コンスタントプールのシンボルの解決処理(resolution) (2) ： 解決処理

--- 
#Under Construction

## 概要(Summary)
(#Under Construction)

## 処理の流れ (概要)(Execution Flows : Summary)
### クラスの解決
<div class="flow-abst"><pre>
constantPoolOopDesc::klass_at()
-&gt; constantPoolOopDesc::klass_at_impl()
   -&gt; (1) 既に一度解決済みかどうかを確認する (= 以前の結果が Constant Pool 中に記録されているかどうか)
          -&gt; constantPoolOopDesc::slot_at()
          -&gt; CPSlot::is_oop()
      (1) 確認結果に応じて分岐
          * 解決済みの場合:
            (この場合は Constant Pool 中の値をリターンするだけ)
            -&gt; CPSlot::get_oop()
          * まだ解決されていない場合: 
            -&gt; (1) クラス名に基づき klassOop を取得する
                   -&gt; SystemDictionary::resolve_or_fail()
                      -&gt; (See: <a href="noIvSV0NZj.html">here</a> for details)
               (1) 解決結果を Constant Pool 中にセットする
                   -&gt; constantPoolOopDesc::klass_at_put()
</pre></div>

### フィールドの解決
<div class="flow-abst"><pre>
LinkResolver::resolve_field(FieldAccessInfo&amp; result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, TRAPS)
-&gt; LinkResolver::resolve_field(FieldAccessInfo&amp; result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, bool update_pool, TRAPS)
   -&gt; (1) 対象フィールドが属するクラスを取得する.
          (クラスが constant pool 中で未解決な場合に解決処理を行うかどうかで2通り)
          * 解決処理を行う場合: 
             -&gt; LinkResolver::resolve_klass()
                -&gt; constantPoolOopDesc::klass_ref_at()
                   -&gt; constantPoolOopDesc::klass_ref_index_at()
                      -&gt; constantPoolOopDesc::impl_klass_ref_index_at()
                   -&gt; constantPoolOopDesc::klass_at()
                      -&gt; (同上)
          * 解決処理を行わない場合: 
            -&gt; LinkResolver::resolve_klass_no_update()
               -&gt; constantPoolOopDesc::klass_ref_at_if_loaded_check()
                   -&gt; constantPoolOopDesc::klass_ref_index_at()
                     -&gt; (同上)
                   -&gt; constantPoolOopDesc::klass_ref_index_at()
                      -&gt; CPSlot::.get_oop()  or  SystemDictionary::find()

      (1) 対象フィールドの情報を取得する
          -&gt; instanceKlass::find_field() 
             -&gt; (Java 仮想マシン仕様に従った順番で探索)
                (1) 自分の中で探索
                    -&gt; instanceKlass::find_local_field()
                (1) インターフェースの中で探索
                    -&gt; instanceKlass::find_interface_field()
                       -&gt; instanceKlass::find_local_field()
                       -&gt; instanceKlass::find_interface_field()  (superinterface に対して再帰呼び出し)
                (1) スーパークラスの中で探索
                    -&gt; instanceKlass::find_field()  (super に対して再帰呼び出し)

      (1) アクセス権をチェックする
          -&gt; LinkResolver::check_field_accessability()

      (1) ロード制約(loading constraints)をチェックする.
          -&gt; SystemDictionary::check_signature_loaders()

      (1) 引数で渡された FieldAccessInfo オブジェクトに解決結果をセット
          -&gt; FieldAccessInfo::set()
</pre></div>

### メソッドの解決
* LinkResolver::resolve_invoke() の処理

<div class="flow-abst"><pre>
LinkResolver::resolve_invoke()
-&gt; バイトコード種別に応じたメソッドで処理を行う
   * invokestatic の場合
     -&gt; LinkResolver::resolve_invokestatic()
        -&gt; (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -&gt; LinkResolver::resolve_pool()
                  -&gt; LinkResolver::resolve_klass()
                     -&gt; constantPoolOopDesc::klass_ref_at()
                        -&gt; constantPoolOopDesc::klass_ref_index_at()
                           -&gt; constantPoolOopDesc::impl_klass_ref_index_at()
                        -&gt; constantPoolOopDesc::klass_at()
                           -&gt; (同上)
           (1) 対象メソッドの解決処理を行う
               -&gt; LinkResolver::resolve_static_call()
                  -&gt; (1) 対象メソッドの情報を取得する
                         -&gt; LinkResolver::linktime_resolve_static_method()
                            -&gt; LinkResolver::resolve_method(methodHandle&amp; resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -&gt; (1) 対象メソッドを, 自分及びスーパークラス階層の中で探索
                                      -&gt; LinkResolver::lookup_method_in_klasses()
                                         -&gt; instanceKlass::uncached_lookup_method()
                                            -&gt; instanceKlass::find_method(Symbol* name, Symbol* signature)
                                               -&gt; instanceKlass::find_method(objArrayOop methods, Symbol* name, Symbol* signature)
                                  (1) (対象メソッドが見つからなければ) インターフェース階層の中で探索
                                      -&gt; LinkResolver::lookup_method_in_interfaces()
                                         -&gt; instanceKlass::lookup_method_in_all_interfaces()
                                            -&gt; Klass::lookup_method()
                                               -&gt; instanceKlass::uncached_lookup_method()
                                                  -&gt; (同上)
                                  (1) (対象メソッドが見つからなければ) 
                                      -&gt; LinkResolver::lookup_implicit_method()
                                         -&gt; 
                                  (1) アクセス権をチェックする
                                      -&gt; LinkResolver::check_method_accessability()
                                  (1) ロード制約(loading constraints)をチェックする
                                      -&gt; SystemDictionary::check_signature_loaders()
                     (1) 引数で渡された CallInfo オブジェクトに, 解決結果をセットする
                         -&gt; CallInfo::set_static()
                            -&gt; CallInfo::set_common() (※)

   * invokespecial の場合
     -&gt; LinkResolver::resolve_invokespecial()
        -&gt; (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -&gt; LinkResolver::resolve_pool()
                  -&gt; (同上)
           (1) 対象メソッドの解決処理を行う
               -&gt; LinkResolver::resolve_special_call()
                  -&gt; (1) 対象メソッドの情報を取得する
                         -&gt; LinkResolver::linktime_resolve_special_method()
                            -&gt; LinkResolver::resolve_method(methodHandle&amp; resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -&gt; (同上)
                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -&gt; LinkResolver::runtime_resolve_special_method()
                            -&gt; CallInfo::set_static()
                               -&gt; (同上)

   * invokevirtual の場合
     -&gt; LinkResolver::resolve_invokevirtual()
        -&gt; (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -&gt; LinkResolver::resolve_pool()
                  -&gt; (同上)
           (1) 対象メソッドの解決処理を行う
               -&gt; LinkResolver::resolve_virtual_call()
                  -&gt; (1) 対象メソッドの情報を取得する
                         -&gt; LinkResolver::linktime_resolve_virtual_method()
                            -&gt; LinkResolver::resolve_method(methodHandle&amp; resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -&gt; (同上)
                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -&gt; LinkResolver::runtime_resolve_virtual_method()
                            -&gt; CallInfo::set_virtual()
                               -&gt; CallInfo::set_common()

   * invokeinterface の場合
     -&gt; LinkResolver::resolve_invokeinterface()
        -&gt; (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -&gt; LinkResolver::resolve_pool()
                  -&gt; (同上)
           (1) 対象メソッドの解決処理を行う
               -&gt; LinkResolver::resolve_interface_call()
                  -&gt; (1) 対象メソッドの情報を取得する
                         -&gt; LinkResolver::linktime_resolve_interface_method()
                            -&gt; LinkResolver::resolve_interface_method(methodHandle&amp; resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -&gt; (1) 対象メソッドを, 自分及びスーパークラス階層の中で探索
                                      -&gt; LinkResolver::lookup_instance_method_in_klasses()
                                         -&gt; instanceKlass::uncached_lookup_method()
                                            -&gt; (同上)
                                  (1) (対象メソッドが見つからなければ) インターフェース階層の中で探索
                                      -&gt; LinkResolver::lookup_method_in_interfaces()
                                         -&gt; (同上)
                                  (1) ロード制約(loading constraints)をチェックする
                                      -&gt; SystemDictionary::check_signature_loaders()

                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -&gt; LinkResolver::runtime_resolve_interface_method()
                            -&gt; CallInfo::set_interface()
                               -&gt; CallInfo::set_common()

   * invokedynamic の場合
     -&gt; LinkResolver::resolve_invokedynamic()
        -&gt; 

(※ -Xcomp 時には, この中で CompileBroker::compile_method() を呼んで JIT コンパイルも実行)
</pre></div>

* ...


### 文字列オブジェクトの解決
<div class="flow-abst"><pre>
constantPoolOopDesc::string_at()
-&gt; constantPoolOopDesc::string_at_impl()
   -&gt; (1) 文字列をインターンし, 対応する文字列オブジェクトを取得する
          -&gt; StringTable::intern()
      (1) 解決結果を Constant Pool 中にセットする
          -&gt; constantPoolOopDesc::string_at_put()
</pre></div>


## 処理の流れ (詳細)(Execution Flows : Details)
### constantPoolOopDesc::klass_at()
See: [here](no7517r_h.html) for details
### constantPoolOopDesc::klass_at_impl()
See: [here](no7517Fbi.html) for details
### constantPoolOopDesc::slot_at()
(#Under Construction)

### CPSlot::is_oop()
(#Under Construction)

### CPSlot::get_oop()
(#Under Construction)

### constantPoolOopDesc::klass_at_put()
(#Under Construction)
See: [here](no19018itU.html) for details

### LinkResolver::resolve_field(FieldAccessInfo& result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, TRAPS)
See: [here](no7882wCV.html) for details
### LinkResolver::resolve_field(FieldAccessInfo& result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, bool update_pool, TRAPS)
See: [here](no78829Mb.html) for details
### LinkResolver::resolve_klass()
See: [here](no19018LOA.html) for details
### constantPoolOopDesc::klass_ref_at()
See: [here](no19018V4f.html) for details
### constantPoolOopDesc::klass_ref_index_at()
See: [here](no19018vTg.html) for details
### constantPoolOopDesc::impl_klass_ref_index_at()
See: [here](no19018vaU.html) for details
### LinkResolver::resolve_klass_no_update()
See: [here](no19018MIT.html) for details
### constantPoolOopDesc::klass_ref_at_if_loaded_check()
See: [here](no19018Bxx.html) for details
### instanceKlass::find_field()
See: [here](no7882KXh.html) for details
### instanceKlass::find_local_field()
See: [here](no7882Xhn.html) for details
### instanceKlass::find_interface_field()
See: [here](no7882krt.html) for details
### LinkResolver::check_field_accessability()
(#Under Construction)

### SystemDictionary::check_signature_loaders()
(#Under Construction)

### FieldAccessInfo::set()
See: [here](no190186dC.html) for details

### LinkResolver::resolve_invoke()
See: [here](no19018NJa.html) for details
### LinkResolver::resolve_invokestatic()
See: [here](no1901801U.html) for details
### LinkResolver::resolve_pool()
See: [here](no19018ORV.html) for details
### LinkResolver::resolve_static_call()
(#Under Construction)
See: [here](no19018r2e.html) for details
### LinkResolver::linktime_resolve_static_method()
See: [here](no19018yRr.html) for details
### LinkResolver::resolve_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
See: [here](no19018zZm.html) for details
### LinkResolver::lookup_method_in_klasses()
See: [here](no19018-cU.html) for details
### instanceKlass::uncached_lookup_method()
See: [here](no19018yMh.html) for details
### instanceKlass::find_method(Symbol* name, Symbol* signature)
See: [here](no190180Ov.html) for details
### instanceKlass::find_method(objArrayOop methods, Symbol* name, Symbol* signature)
See: [here](no19018ziE.html) for details
### LinkResolver::lookup_method_in_interfaces()
See: [here](no19018cDl.html) for details
### instanceKlass::lookup_method_in_all_interfaces()
See: [here](no190182lZ.html) for details
### Klass::lookup_method()
See: [here](no190183tU.html) for details
### LinkResolver::lookup_implicit_method()
(#Under Construction)

### LinkResolver::check_method_accessability()
(#Under Construction)

### SystemDictionary::check_signature_loaders()
(#Under Construction)

### CallInfo::set_static()
See: [here](no1901813E.html) for details
### CallInfo::set_common()
See: [here](no19018Q_v.html) for details

### LinkResolver::resolve_invokespecial()
See: [here](no19018VzV.html) for details
### LinkResolver::resolve_special_call()
See: [here](no19018sqm.html) for details
### LinkResolver::linktime_resolve_special_method()
See: [here](no1901868n.html) for details
### LinkResolver::runtime_resolve_special_method()
See: [here](no19018ghn.html) for details
### LinkResolver::resolve_invokevirtual()
See: [here](no19018wB1.html) for details
### LinkResolver::resolve_virtual_call()
See: [here](no19018v7j.html) for details
### LinkResolver::linktime_resolve_virtual_method()
See: [here](no190188Me.html) for details
### LinkResolver::runtime_resolve_virtual_method()
(#Under Construction)
See: [here](no19018UaS.html) for details
### CallInfo::set_virtual()
See: [here](no19018IRT.html) for details
### LinkResolver::resolve_invokeinterface()
See: [here](no19018k41.html) for details
### LinkResolver::resolve_interface_call()
See: [here](no19018w8q.html) for details
### LinkResolver::linktime_resolve_interface_method()
See: [here](no19018JsA.html) for details
### LinkResolver::resolve_interface_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
See: [here](no19018Ndq.html) for details
### LinkResolver::lookup_instance_method_in_klasses()
(#Under Construction)
See: [here](no19018OsZ.html) for details
### LinkResolver::runtime_resolve_interface_method()
See: [here](no19018-XK.html) for details
### CallInfo::set_interface()
See: [here](no19018isT.html) for details
### LinkResolver::resolve_invokedynamic()
(#Under Construction)

### constantPoolOopDesc::string_at()
See: [here](no19018dJB.html) for details
### constantPoolOopDesc::string_at_impl()
See: [here](no19018e8f.html) for details
### StringTable::intern()
(#Under Construction)

### constantPoolOopDesc::string_at_put()
(#Under Construction)
See: [here](no19018TJE.html) for details






