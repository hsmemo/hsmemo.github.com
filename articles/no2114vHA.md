---
layout: default
title: (unrecognied function)(FIXME!)
---
[Top](../index.html)

--- 
### 定義場所(file name)
jdk/src/solaris/classes/com/sun/management/OSMBeanFactory.java

### 名前(function name)
```
    public static synchronized OperatingSystemMXBean
        getOperatingSystemMXBean(VMManagement jvm) {
```

### 本体部(body)
```
  {- -------------------------------------------
  (1) 新しい com.sun.management.UnixOperatingSystem のインスタンスを作ってリターンする
      ---------------------------------------- -}
	
	        if (osMBean == null) {
	            osMBean = new UnixOperatingSystem(jvm);
	        }
	        return (OperatingSystemMXBean) osMBean;
	    }
	
```


