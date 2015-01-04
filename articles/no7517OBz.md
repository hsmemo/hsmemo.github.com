---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/share/classes/java/lang/Class.java

### 名前(function name)
```
    private static Method searchMethods(Method[] methods,
                                        String name,
                                        Class<?>[] parameterTypes)
    {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) (変数宣言など)
      ---------------------------------------- -}

	        Method res = null;
	        String internedName = name.intern();

  {- -------------------------------------------
  (1) methods 引数で指定された配列を辿り, 
      メソッド名が name 引数と等しく, かつ引数の型情報が parameterTypes 引数に等しい
      Method オブジェクトを探す.
  
      (なお, 該当するメソッドが複数あった場合は, ...)
      ---------------------------------------- -}

	        for (int i = 0; i < methods.length; i++) {
	            Method m = methods[i];
	            if (m.getName() == internedName
	                && arrayContentsEq(parameterTypes, m.getParameterTypes())
	                && (res == null
	                    || res.getReturnType().isAssignableFrom(m.getReturnType())))
	                res = m;
	        }
	
  {- -------------------------------------------
  (1) 結果をリターン
      ---------------------------------------- -}

	        return (res == null ? res : getReflectionFactory().copyMethod(res));
	    }
	
```


