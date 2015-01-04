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
```
constantPoolOopDesc::klass_at()
-> constantPoolOopDesc::klass_at_impl()
   -> (1) 既に一度解決済みかどうかを確認する (= 以前の結果が Constant Pool 中に記録されているかどうか)
          -> constantPoolOopDesc::slot_at()
          -> CPSlot::is_oop()
      (1) 確認結果に応じて分岐
          * 解決済みの場合:
            (この場合は Constant Pool 中の値をリターンするだけ)
            -> CPSlot::get_oop()
          * まだ解決されていない場合: 
            -> (1) クラス名に基づき klassOop を取得する
                   -> SystemDictionary::resolve_or_fail()
                      -> (See: [here](noIvSV0NZj.html) for details)
               (1) 解決結果を Constant Pool 中にセットする
                   -> constantPoolOopDesc::klass_at_put()
```

### フィールドの解決
```
LinkResolver::resolve_field(FieldAccessInfo& result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, TRAPS)
-> LinkResolver::resolve_field(FieldAccessInfo& result, constantPoolHandle pool, int index, Bytecodes::Code byte, bool check_only, bool update_pool, TRAPS)
   -> (1) 対象フィールドが属するクラスを取得する.
          (クラスが constant pool 中で未解決な場合に解決処理を行うかどうかで2通り)
          * 解決処理を行う場合: 
             -> LinkResolver::resolve_klass()
                -> constantPoolOopDesc::klass_ref_at()
                   -> constantPoolOopDesc::klass_ref_index_at()
                      -> constantPoolOopDesc::impl_klass_ref_index_at()
                   -> constantPoolOopDesc::klass_at()
                      -> (同上)
          * 解決処理を行わない場合: 
            -> LinkResolver::resolve_klass_no_update()
               -> constantPoolOopDesc::klass_ref_at_if_loaded_check()
                   -> constantPoolOopDesc::klass_ref_index_at()
                     -> (同上)
                   -> constantPoolOopDesc::klass_ref_index_at()
                      -> CPSlot::.get_oop()  or  SystemDictionary::find()

      (1) 対象フィールドの情報を取得する
          -> instanceKlass::find_field() 
             -> (Java 仮想マシン仕様に従った順番で探索)
                (1) 自分の中で探索
                    -> instanceKlass::find_local_field()
                (1) インターフェースの中で探索
                    -> instanceKlass::find_interface_field()
                       -> instanceKlass::find_local_field()
                       -> instanceKlass::find_interface_field()  (superinterface に対して再帰呼び出し)
                (1) スーパークラスの中で探索
                    -> instanceKlass::find_field()  (super に対して再帰呼び出し)

      (1) アクセス権をチェックする
          -> LinkResolver::check_field_accessability()

      (1) ロード制約(loading constraints)をチェックする.
          -> SystemDictionary::check_signature_loaders()

      (1) 引数で渡された FieldAccessInfo オブジェクトに解決結果をセット
          -> FieldAccessInfo::set()
```

### メソッドの解決
* LinkResolver::resolve_invoke() の処理

```
LinkResolver::resolve_invoke()
-> バイトコード種別に応じたメソッドで処理を行う
   * invokestatic の場合
     -> LinkResolver::resolve_invokestatic()
        -> (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -> LinkResolver::resolve_pool()
                  -> LinkResolver::resolve_klass()
                     -> constantPoolOopDesc::klass_ref_at()
                        -> constantPoolOopDesc::klass_ref_index_at()
                           -> constantPoolOopDesc::impl_klass_ref_index_at()
                        -> constantPoolOopDesc::klass_at()
                           -> (同上)
           (1) 対象メソッドの解決処理を行う
               -> LinkResolver::resolve_static_call()
                  -> (1) 対象メソッドの情報を取得する
                         -> LinkResolver::linktime_resolve_static_method()
                            -> LinkResolver::resolve_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -> (1) 対象メソッドを, 自分及びスーパークラス階層の中で探索
                                      -> LinkResolver::lookup_method_in_klasses()
                                         -> instanceKlass::uncached_lookup_method()
                                            -> instanceKlass::find_method(Symbol* name, Symbol* signature)
                                               -> instanceKlass::find_method(objArrayOop methods, Symbol* name, Symbol* signature)
                                  (1) (対象メソッドが見つからなければ) インターフェース階層の中で探索
                                      -> LinkResolver::lookup_method_in_interfaces()
                                         -> instanceKlass::lookup_method_in_all_interfaces()
                                            -> Klass::lookup_method()
                                               -> instanceKlass::uncached_lookup_method()
                                                  -> (同上)
                                  (1) (対象メソッドが見つからなければ) 
                                      -> LinkResolver::lookup_implicit_method()
                                         -> 
                                  (1) アクセス権をチェックする
                                      -> LinkResolver::check_method_accessability()
                                  (1) ロード制約(loading constraints)をチェックする
                                      -> SystemDictionary::check_signature_loaders()
                     (1) 引数で渡された CallInfo オブジェクトに, 解決結果をセットする
                         -> CallInfo::set_static()
                            -> CallInfo::set_common() (※)

   * invokespecial の場合
     -> LinkResolver::resolve_invokespecial()
        -> (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -> LinkResolver::resolve_pool()
                  -> (同上)
           (1) 対象メソッドの解決処理を行う
               -> LinkResolver::resolve_special_call()
                  -> (1) 対象メソッドの情報を取得する
                         -> LinkResolver::linktime_resolve_special_method()
                            -> LinkResolver::resolve_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -> (同上)
                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -> LinkResolver::runtime_resolve_special_method()
                            -> CallInfo::set_static()
                               -> (同上)

   * invokevirtual の場合
     -> LinkResolver::resolve_invokevirtual()
        -> (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -> LinkResolver::resolve_pool()
                  -> (同上)
           (1) 対象メソッドの解決処理を行う
               -> LinkResolver::resolve_virtual_call()
                  -> (1) 対象メソッドの情報を取得する
                         -> LinkResolver::linktime_resolve_virtual_method()
                            -> LinkResolver::resolve_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -> (同上)
                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -> LinkResolver::runtime_resolve_virtual_method()
                            -> CallInfo::set_virtual()
                               -> CallInfo::set_common()

   * invokeinterface の場合
     -> LinkResolver::resolve_invokeinterface()
        -> (1) 対象メソッドが属するクラス(及びその他の情報)を取得する
               -> LinkResolver::resolve_pool()
                  -> (同上)
           (1) 対象メソッドの解決処理を行う
               -> LinkResolver::resolve_interface_call()
                  -> (1) 対象メソッドの情報を取得する
                         -> LinkResolver::linktime_resolve_interface_method()
                            -> LinkResolver::resolve_interface_method(methodHandle& resolved_method, KlassHandle resolved_klass, Symbol* method_name, Symbol* method_signature, KlassHandle current_klass, bool check_access, TRAPS)
                               -> (1) 対象メソッドを, 自分及びスーパークラス階層の中で探索
                                      -> LinkResolver::lookup_instance_method_in_klasses()
                                         -> instanceKlass::uncached_lookup_method()
                                            -> (同上)
                                  (1) (対象メソッドが見つからなければ) インターフェース階層の中で探索
                                      -> LinkResolver::lookup_method_in_interfaces()
                                         -> (同上)
                                  (1) ロード制約(loading constraints)をチェックする
                                      -> SystemDictionary::check_signature_loaders()

                     (1) 解決結果をチェックし, 問題なければ引数で渡された CallInfo オブジェクトに結果をセットする
                         -> LinkResolver::runtime_resolve_interface_method()
                            -> CallInfo::set_interface()
                               -> CallInfo::set_common()

   * invokedynamic の場合
     -> LinkResolver::resolve_invokedynamic()
        -> 

(※ -Xcomp 時には, この中で CompileBroker::compile_method() を呼んで JIT コンパイルも実行)
```

* ...


### 文字列オブジェクトの解決
```
constantPoolOopDesc::string_at()
-> constantPoolOopDesc::string_at_impl()
   -> (1) 文字列をインターンし, 対応する文字列オブジェクトを取得する
          -> StringTable::intern()
      (1) 解決結果を Constant Pool 中にセットする
          -> constantPoolOopDesc::string_at_put()
```


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






