---
layout: default
title: Class のロード/リンク/初期化 ： ロード処理 (3) ： クラスファイルのパース処理 
---
[Up](nohUsh5oi7.html) [Top](../index.html)

#### Class のロード/リンク/初期化 ： ロード処理 (3) ： クラスファイルのパース処理 

--- 
## 概要(Summary)
クラスファイルのパース処理は ClassFileParser::parseClassFile() に実装されている.

ClassFileParser::parseClassFile() では, 
クラスファイルの要素を先頭から読んでいき,
各要素がクラスファイルの仕様に従っているかどうかを確認するとともに, 
それぞれに対応するデータ構造を生成していく.
最終的には, そのクラスを表す klassOop が返される.

## 備考(Notes)
生成された klassOop は, 
ClassFileParser::parseClassFile() の呼び出し元が SystemDictionary に登録する必要がある
(ClassFileParser::parseClassFile() は SystemDictionary への登録はやってくれないため).


```cpp
    ((cite: hotspot/src/share/vm/classfile/classFileParser.hpp))
      // Parse .class file and return new klassOop. The klassOop is not hooked up
      // to the system dictionary or any other structures, so a .class file can
      // be loaded several times if desired.
      // The system dictionary hookup is done by the caller.
      //
      // "parsed_name" is updated by this method, and is the name found
      // while parsing the stream.
      instanceKlassHandle parseClassFile(Symbol* name,
```

## 処理の流れ (概要)(Execution Flows : Summary)
<div class="flow-abst"><pre>
ClassFileParser::parseClassFile()
-&gt; (1) ClassFileLoadHook イベントの処理
       -&gt; JvmtiExport::post_class_file_load_hook()
          -&gt; (略) (See: <a href="no2935WjX.html">here</a> for details)

   (1) 対象のクラスに対する verify が必要かどうかを確認.
       -&gt; Verifier::should_verify_for()

   (1) クラスファイルの magic number や version number をチェックする.
       -&gt; ClassFileParser::is_supported_version()

   (1) 「いくつかの broken な 1.1 API を 1.2 でも動かすために, verify を relaxed させるべきかどうか」を調べる
       -&gt; Verifier::relax_verify_for()

   (1) コンスタントプール情報を読み込む
       -&gt; ClassFileParser::parse_constant_pool()
          -&gt; (1) constant_pool_count 情報(長さ情報) を読み込む
             (1) 長さ分だけの constantPoolOop を確保し, 初期化する
                 -&gt; oopFactory::new_constantPool()
             (1) constant pool 内の各エントリを読み込み, constantPoolOop オブジェクトに設定する
                 -&gt; ClassFileParser::parse_constant_pool_entries()
                    -&gt; constantPoolOopDesc::klass_index_at_put()
                    -&gt; constantPoolOopDesc::field_at_put()
                    -&gt; constantPoolOopDesc::method_at_put()
                    -&gt; constantPoolOopDesc::interface_method_at_put()
                    -&gt; constantPoolOopDesc::string_index_at_put()
                    -&gt; constantPoolOopDesc::method_type_index_at_put()
                    -&gt; constantPoolOopDesc::invoke_dynamic_at_put()
                    -&gt; constantPoolOopDesc::int_at_put()
                    -&gt; constantPoolOopDesc::float_at_put()
                    -&gt; constantPoolOopDesc::long_at_put()
                    -&gt; constantPoolOopDesc::double_at_put()
                    -&gt; constantPoolOopDesc::name_and_type_at_put()
                    -&gt; constantPoolOopDesc::symbol_at_put()
                    -&gt; SymbolTable::new_symbols()
             (1) 内容を vefify し, String 部分や Class 部分を Unresolved タグに書き換える

   (1) access_flags 情報を読み込み, 値をチェックする.
       -&gt; ClassFileParser::verify_legal_class_modifiers()
       -&gt; AccessFlags::set_flags()

   (1) this_class 情報を読み込み, 値をチェックする. 
       -&gt; (1) index の値が, Constant Pool 内に収まっており, unresolved な klass を指していることをチェック.
              -&gt; valid_cp_range()
              -&gt; constantTag::is_unresolved_klass()
              -&gt; ClassFileParser::check_property()
          (1) パースした constant pool から this_class のクラス名を取得.
              -&gt; constantPoolOopDesc::unresolved_klass_at()
          (1) verify の必要があれば, this_class が array type になっていないことを確認.
              -&gt; ClassFileParser::guarantee_property()
          (1) this_class のクラス名をチェック (null だったり要求されていたクラス名と違っていたら, NoClassDefFoundError)

   (1) super_class 情報を読み込み, 値をチェックする.
       -&gt; (1) super_class_index が 0 だった場合は, java_lang_Object かどうかを確認.
          (1) super_class_index が 0 でなければ, その index 値が Constant Pool 内に収まっており class を指していること, また array ではないことを確認.
              -&gt; ClassFileParser::is_klass_reference()
              -&gt; ClassFileParser::check_property()
              -&gt; ClassFileParser::guarantee_property()

   (1) 実装しているインターフェース情報を読み込む (実装したインターフェースの個数, 及びそのインターフェース情報).
       -&gt; ClassFileParser::parse_interfaces()
          -&gt; ClassFileParser::check_property()
          -&gt; SystemDictionary::resolve_super_or_fail()

   (1) フィールド情報を読み込む (フィールドの個数, 及び実際のフィールド情報) (= field_info 構造体を全て読み出す)
       -&gt; ClassFileParser::parse_fields()
          -&gt; (1) フィールドの個数情報を読み込む
             (1) 長さ(× u2 7個)分の typeArrayOop を確保し, 初期化する
                 -&gt; oopFactory::new_permanent_shortArray()
             (1) 各フィールド情報を読み込み, typeArrayOop 内に設定する
                 -&gt; (1) access_flags を読み込み, おかしな設定ではないかどうか確認する.
                        -&gt; ClassFileParser::verify_legal_field_modifiers()
                    (1) name_index を読み込み, その index 値が Constant Pool 内に収まっており, utf8_info を指していることを確認する.
                    (1) フィールド名として妥当な utf8 文字列かどうかを確認する.
                        -&gt; ClassFileParser::verify_legal_field_name()
                    (1) signature_index (JVMS では descriptor_index) を読み込み, その index 値が Constant Pool 内に収まっており, utf8_info を指していることを確認する.
                    (1) フィールドの型情報(フィールド・ディスクリプタ)として妥当な utf8 文字列かどうかを確認する.
                        -&gt; ClassFileParser::verify_legal_field_signature()
                    (1) フィールドのアトリビュート情報のパース
                        -&gt; ClassFileParser::parse_field_attributes()
                           -&gt; ClassFileParser::verify_constantvalue()
                           -&gt; 
                    (1) フィールド・ディスクリプタ文字列を基に, フィールドの allocation type (FieldAllocationType) を決定する.
                        -&gt; ConstantPoolOop::basic_type_for_signature_at()

   (1) メソッド情報を読み込む (メソッドの個数, 及び実際のメソッド情報) (= method_info 構造体を全て読み出す)
       (最終的に読み取った情報は全て methodOop にまとめる. ただし, アノテーションは後から追加の情報として値が返される)
       -&gt; ClassFileParser::parse_methods()
          -&gt; (1) メソッドの個数情報を読み込む
             (1) 個数分の objArrayOop を確保し, 初期化する
                 -&gt; oopFactory::new_permanent_shortArray()
             (1) 各メソッド情報を読み込み, objArrayOop 内に設定する
                 -&gt; ClassFileParser::parse_method()
                    -&gt; (1) access_flags を読み込む
                       (1) name_index を読み込み, その index 値が Constant Pool 内に収まっており, utf8_info を指していることを確認する.
                       (1) メソッド名として妥当な utf8 文字列かどうかを確認する.
                           -&gt; ClassFileParser::verify_legal_method_name()
                       (1) signature_index (JVMS では descriptor_index) を読み込み, その index 値が Constant Pool 内に収まっており, utf8_info を指していることを確認する.
                       (1) access_flags が, おかしな設定ではないかどうか確認する.
                           -&gt; ClassFileParser::verify_legal_method_modifiers()
                       (1) メソッドの型情報(メソッド・ディスクリプタ)として妥当な utf8 文字列かどうかを確認する.
                           -&gt; ClassFileParser::verify_legal_method_signature()
                       (1) メソッドのアトリビュート情報のパース
                           -&gt; ClassFileParser::parse_exception_table()
                           -&gt; ClassFileParser::parse_checked_exceptions()
                           -&gt; 
                       (1) methodOop を生成する
                           -&gt; oopFactory::new_method()

   (1) スーパークラスのチェックを行う.
       (スーパークラスを再帰的にロードする. クラスローダーが違うなどのエラーがあれば, このクラスの処理まで戻って例外発生)
       -&gt; (1) もしスーパークラスが存在し, かつまだロードされてなければ, ロードする.
              -&gt; SystemDictionary::resolve_super_or_fail()
          (1) super_klass が null でない(= java/lang/Object ではない?#TODO) 場合は,
              super_klass がインターフェースではなく, final クラスでもないことを確認.
              (もしインターフェースであれば IncompatibleClassChangeError, final クラスであれば VerifyError を出す)

   (1) 実装しているインターフェースの推移的平方の数を計算し, その数にあった objArrayHandle を作成する.
       -&gt; ClassFileParser::compute_transitive_interfaces()

   (1) 全メソッドを名前順でソートし, vtable 中での index を割り振る.
       -&gt; ClassFileParser::sort_methods()
          -&gt; methodOopDesc::sort_methods()

   (1) ClassFileParser::parse_methods() の処理中で発見した「クラスのフラグに追加した方がいいフラグ」を access_flags に追加する.
       -&gt; AccessFlags::add_promoted_flags()

   (1) vtable, itable の大きさと miranda method の個数を計算する.
       -&gt; klassVtable::compute_vtable_size_and_num_mirandas()

   (1) 各フィールドについて, オブジェクト内でのオフセットを決める. (#TODO)
       -&gt; 

   (1) oopmap の大きさ(= OopMapBlock オブジェクトが占める大きさ)を計算する
       -&gt; ClassFileParser::compute_oop_map_count()

   (1) reference type (= java.lang.ref.Reference クラスのサブクラスかどうか) を調べる

   (1) パース結果を格納するための klassOop を確保する.
       -&gt; oopFactory::new_instanceKlass()

   (1) 確保した klassOop 内にパース結果の情報を書き込んでいく.
       -&gt; (1) ...

          (1) クラスファイルのアトリビュートをパースする
              -&gt; ClassFileParser::parse_classfile_attributes()
          (1) oopmap 領域に OopMapBlock オブジェクトを書き込む
              -&gt; ClassFileParser::fill_oop_maps()

   (1) 正しいクラスになっていることをチェックする.
       -&gt; ClassFileParser::check_super_class_access()
       -&gt; ClassFileParser::check_super_interface_access()
       -&gt; ClassFileParser::check_final_method_override()
       -&gt; ClassFileParser::check_illegal_static_method()

   (1) mirror オブジェクト (= Java レベルでのクラスオブジェクト) を生成する.
       -&gt; java_lang_Class::create_mirror()
</pre></div>

## 処理の流れ (詳細)(Execution Flows : Details)
### ClassFileParser::parseClassFile()
(#Under Construction)
See: [here](no4230X6P.html) for details
### Verifier::should_verify_for()
(#Under Construction)
See: [here](no2099kuD.html) for details
### ClassFileParser::is_supported_version()
(#Under Construction)
See: [here](no2099_1u.html) for details
### Verifier::relax_verify_for()
See: [here](no20994nJ.html) for details
### ClassFileParser::parse_constant_pool()
(#Under Construction)
See: [here](no4230aSB.html) for details
### oopFactory::new_constantPool()
See: [here](no42300mN.html) for details
### constantPoolKlass::allocate()
See: [here](no4230BxT.html) for details
### ClassFileParser::parse_constant_pool_entries()
(#Under Construction)
See: [here](no4230O7Z.html) for details
### constantPoolOopDesc::klass_index_at_put()
See: [here](no2099OQr.html) for details
### constantPoolOopDesc::field_at_put()
See: [here](no2099auG.html) for details
### constantPoolOopDesc::method_at_put()
See: [here](no2099n4M.html) for details
### constantPoolOopDesc::interface_method_at_put()
See: [here](no2099OXf.html) for details
### constantPoolOopDesc::string_index_at_put()
See: [here](no209911x.html) for details
### constantPoolOopDesc::method_handle_index_at_put()
See: [here](no2099Pfa.html) for details
### constantPoolOopDesc::method_type_index_at_put()
See: [here](no2099p6a.html) for details
### constantPoolOopDesc::invoke_dynamic_at_put()
See: [here](no20992Eh.html) for details
### constantPoolOopDesc::int_at_put()
See: [here](no2099DPn.html) for details
### constantPoolOopDesc::float_at_put()
See: [here](no2099djz.html) for details
### constantPoolOopDesc::long_at_put()
See: [here](no2099QZt.html) for details
### constantPoolOopDesc::double_at_put()
See: [here](no2099PtC.html) for details
### constantPoolOopDesc::name_and_type_at_put()
See: [here](no2099DWb.html) for details
### constantPoolOopDesc::symbol_at_put()
See: [here](no2099c3I.html) for details
### SymbolTable::new_symbols()
(#Under Construction)


### ClassFileParser::verify_legal_class_modifiers()
See: [here](no4230b-r.html) for details
### AccessFlags::set_flags()
(#Under Construction)


### valid_cp_range()
(#Under Construction)

### constantTag::is_unresolved_klass()
(#Under Construction)

### ClassFileParser::check_property()
(#Under Construction)

### constantPoolOopDesc::unresolved_klass_at()
(#Under Construction)

### ClassFileParser::guarantee_property()
(#Under Construction)

### ClassFileParser::is_klass_reference()
(#Under Construction)


### ClassFileParser::parse_interfaces()
See: [here](no171199MB.html) for details

### ClassFileParser::parse_fields()
See: [here](no17119KXH.html) for details
### ClassFileParser::verify_legal_field_modifiers()
(#Under Construction)

### ClassFileParser::verify_legal_field_name()
(#Under Construction)

### ClassFileParser::verify_legal_field_signature()
(#Under Construction)

### ClassFileParser::parse_field_attributes()
See: [here](no17119-_f.html) for details
### ClassFileParser::verify_constantvalue()
(#Under Construction)

### ConstantPoolOop::basic_type_for_signature_at()
(#Under Construction)


### ClassFileParser::parse_methods()
See: [here](no17119YUs.html) for details
### ClassFileParser::parse_method()
See: [here](no17119ley.html) for details
### ClassFileParser::verify_legal_method_name()
(#Under Construction)

### ClassFileParser::verify_legal_method_modifiers()
(#Under Construction)

### ClassFileParser::verify_legal_method_signature()
(#Under Construction)

### ClassFileParser::parse_exception_table()
See: [here](no3059e3p.html) for details

### ClassFileParser::sort_methods()
See: [here](no17119LKm.html) for details

### ClassFileParser::compute_oop_map_count()
See: [here](no2935kwO.html) for details
### ClassFileParser::fill_oop_maps()
See: [here](no2935x6U.html) for details






