---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
hotspot/src/share/vm/oops/methodOop.cpp

### 名前(function name)
```
int  methodOopDesc::fast_exception_handler_bci_for(KlassHandle ex_klass, int throw_bci, TRAPS) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  // exception table holds quadruple entries of the form (beg_bci, end_bci, handler_bci, klass_index)
	  const int beg_bci_offset     = 0;
	  const int end_bci_offset     = 1;
	  const int handler_bci_offset = 2;
	  const int klass_index_offset = 3;
	  const int entry_size         = 4;

  {- -------------------------------------------
  (1) (変数宣言など)
      (table は, クラスファイルからパースした例外ハンドラテーブル)
      ---------------------------------------- -}

	  // access exception table
	  typeArrayHandle table (THREAD, constMethod()->exception_table());
	  int length = table->length();

  {- -------------------------------------------
  (1) (assert)
      ---------------------------------------- -}

	  assert(length % entry_size == 0, "exception table format has changed");

  {- -------------------------------------------
  (1) (以下の for ループで, 例外ハンドラテーブルを先頭から順に辿って対応するハンドラがないか調べる.
       先頭から順に辿るのは JavaVM 仕様上の要請)
      ---------------------------------------- -}

	  // iterate through all entries sequentially

  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	  constantPoolHandle pool(THREAD, constants());

  {- -------------------------------------------
  (1) (例外ハンドラテーブルを先頭から順に辿り, 対応するハンドラがないか調べる.
       見つかったら, 対応するハンドラのエントリポイント(を表す BCI)をリターン)
      ---------------------------------------- -}

	  for (int i = 0; i < length; i += entry_size) {

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	    int beg_bci = table->int_at(i + beg_bci_offset);
	    int end_bci = table->int_at(i + end_bci_offset);
	    assert(beg_bci <= end_bci, "inconsistent exception table");

    {- -------------------------------------------
  (1.1) 例外が投げられた BCI が, 現在調べているハンドラの対象範囲内かどうかをチェック.
        ---------------------------------------- -}

	    if (beg_bci <= throw_bci && throw_bci < end_bci) {
	      // exception handler bci range covers throw_bci => investigate further

    {- -------------------------------------------
  (1.1) (変数宣言など)
        ---------------------------------------- -}

	      int handler_bci = table->int_at(i + handler_bci_offset);
	      int klass_index = table->int_at(i + klass_index_offset);

    {- -------------------------------------------
  (1.1) 投げられた例外オブジェクトが, 現在調べているハンドラの捕捉対象かどうかをチェック.
        以下の条件のどれかに当てはまれば, 対応するハンドラが見つかったことになり, ここでリターン.
        * 例外ハンドラの catch_type 情報(以下の klass_index) が 0
          これは finally を表す (常に該当する).
        * 
        * 投げられた例外オブジェクトが, 例外ハンドラの捕捉対象(catch_type 情報)のサブクラス
        ---------------------------------------- -}

	      if (klass_index == 0) {
	        return handler_bci;
	      } else if (ex_klass.is_null()) {
	        return handler_bci;
	      } else {
	        // we know the exception class => get the constraint class
	        // this may require loading of the constraint class; if verification
	        // fails or some other exception occurs, return handler_bci
	        klassOop k = pool->klass_at(klass_index, CHECK_(handler_bci));
	        KlassHandle klass = KlassHandle(THREAD, k);
	        assert(klass.not_null(), "klass not loaded");
	        if (ex_klass->is_subtype_of(klass())) {
	          return handler_bci;
	        }
	      }
	    }
	  }
	
  {- -------------------------------------------
  (1) 対応する例外ハンドラが見つからなかった場合は, -1 をリターン
      ---------------------------------------- -}

	  return -1;
	}
	
```


