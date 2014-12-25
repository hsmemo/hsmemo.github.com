---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 概要
---
[Up](no2114S_x.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 概要

--- 
## 概要(Summary)
JMM 機能では JSR-174 で規定されている Platform MXBean インターフェースの実装を提供している. 
また, JSR-174 には規定されていない HotSpot 独自の Platform MXBean も提供している.

以下, インターフェースと実装クラスの双方を説明する.

### HotSpot が提供している Platform MXBean インターフェース
HotSpot では以下の3種類のパッケージで Platform MXBean インターフェースを提供している.

  * java.lang.management
    
    JSR-174 で規定されているインターフェース.
   
  * com.sun.management
    
    HotSpot 独自のインターフェース.
   
  * sun.management
    
    HotSpot 独自の (非公式な) インターフェース. 通達無くインターフェース仕様が変更される可能性がある.

具体的なインターフェース名の一覧は以下の通り.

  * java.lang.management.ClassLoadingMXBean
  * java.lang.management.CompilationMXBean
  * java.lang.management.MemoryMXBean
  * java.lang.management.MemoryPoolMXBean
  * java.lang.management.MemoryManagerMXBean
      * java.lang.management.GarbageCollectorMXBean
          * com.sun.management.GarbageCollectorMXBean    (HotSpot 独自)
  * java.lang.management.OperatingSystemMXBean
      * com.sun.management.OperatingSystemMXBean         (HotSpot 独自)
          * com.sun.management.UnixOperatingSystemMXBean (HotSpot 独自)
  * java.lang.management.RuntimeMXBean
  * java.lang.management.ThreadMXBean
      * com.sun.management.ThreadMXBean                  (HotSpot 独自)
  
  * com.sun.management.HotSpotDiagnosticMXBean           (HotSpot 独自)
  
  * sun.management.HotspotClassLoadingMBean              (HotSpot 独自, 非公式)
  * sun.management.HotspotCompilationMBean               (HotSpot 独自, 非公式)
  * sun.management.HotspotInternalMBean                  (HotSpot 独自, 非公式)
  * sun.management.HotspotMemoryMBean                    (HotSpot 独自, 非公式)
  * sun.management.HotspotRuntimeMBean                   (HotSpot 独自, 非公式)
  * sun.management.HotspotThreadMBean                    (HotSpot 独自, 非公式)

### HotSpot が提供している Platform MXBean 実装
Platform MXBean インターフェースを実装したクラスは sun.management パッケージに格納されている.

  (参考: <http://openjdk.java.net/groups/serviceability/svcjdk.html>
        "The platform MBeans are implemented by classes in sun.management.")

具体的な実装クラスの一覧は以下の通り.

  * sun.management.ClassLoadingImpl
  * sun.management.CompilationImpl
  * sun.management.MemoryImpl
  * sun.management.MemoryPoolImpl
  * sun.management.MemoryManagerImpl
  * sun.management.GarbageCollectorImpl
      * (Solaris の場合) com.sun.management.UnixOperatingSystem  (extends sun.management.OperatingSystemImpl)
      * (Windows の場合) com.sun.management.OperatingSystem      (extends sun.management.OperatingSystemImpl)
  * sun.management.RuntimeImpl
  * sun.management.ThreadImpl
  
  * sun.management.HotSpotDiagnostic
  
  * sun.management.HotspotClassLoading
  * sun.management.HotspotCompilation
  * sun.management.HotspotInternal
  * sun.management.HotspotMemory
  * sun.management.HotspotRuntime
  * sun.management.HotspotThread

## 備考(Notes)
con.sum.management パッケージ以下には, 上記の Platform MXBean クラス群を補佐するためのクラス群も定義されている.
具体的なクラス名は以下の通り.

  * com.sun.management.GcInfo
  * com.sun.management.VMOption
  * com.sun.management.GarbageCollectionNotificationInfo


## 参考(for your information)
com.sun.management 以下の HotSpot 独自のインターフェースの一覧:

* com.sun.management.HotSpotDiagnosticMXBean

```java
    ((cite: jdk/src/share/classes/com/sun/management/HotSpotDiagnosticMXBean.java))
    package com.sun.management;
    ...
    /**
     * Diagnostic management interface for the HotSpot Virtual Machine.
     * The diagnostic MBean is registered to the platform MBeanServer
     * as are other platform MBeans.
     *
     * <p>The <tt>ObjectName</tt> for uniquely identifying the diagnostic
     * MXBean within an MBeanServer is:
     * <blockquote>
     *    <tt>com.sun.management:type=HotSpotDiagnostic</tt>
     * </blockquote>
    .*
     * It can be obtained by calling the
     * {@link PlatformManagedObject#getObjectName} method.
     *
     * @see ManagementFactory#getPlatformMXBeans(Class)
     */
    public interface HotSpotDiagnosticMXBean extends PlatformManagedObject {
```

* com.sun.management.GarbageCollectorMXBean

```java
    ((cite: jdk/src/share/classes/com/sun/management/GarbageCollectorMXBean.java))
    package com.sun.management;
    ...
    /**
     * Platform-specific management interface for a garbage collector
     * which performs collections in cycles.
     *
     * <p> This platform extension is only available to the garbage
     * collection implementation that supports this extension.
     *
     * @author  Mandy Chung
     * @since   1.5
     */
    public interface GarbageCollectorMXBean
        extends java.lang.management.GarbageCollectorMXBean {
```

* com.sun.management.OperatingSystemMXBean

```java
    ((cite: jdk/src/share/classes/com/sun/management/OperatingSystemMXBean.java))
    package com.sun.management;
    ...
    /**
     * Platform-specific management interface for the operating system
     * on which the Java virtual machine is running.
     *
     * <p>
     * The <tt>OperatingSystemMXBean</tt> object returned by
     * {@link java.lang.management.ManagementFactory#getOperatingSystemMXBean()}
     * is an instance of the implementation class of this interface
     * or {@link UnixOperatingSystemMXBean} interface depending on
     * its underlying operating system.
     *
     * @author  Mandy Chung
     * @since   1.5
     */
    public interface OperatingSystemMXBean extends
        java.lang.management.OperatingSystemMXBean {
```

* com.sun.management.UnixOperatingSystemMXBean

```java
    ((cite: jdk/src/share/classes/com/sun/management/UnixOperatingSystemMXBean.java))
    package com.sun.management;
    ...
    /**
     * Platform-specific management interface for the Unix
     * operating system on which the Java virtual machine is running.
     *
     * @author  Mandy Chung
     * @since   1.5
     */
    public interface UnixOperatingSystemMXBean extends
        com.sun.management.OperatingSystemMXBean {
```

* com.sun.management.ThreadMXBean

```java
    ((cite: jdk/src/share/classes/com/sun/management/ThreadMXBean.java))
    package com.sun.management;
    ...
    /**
     * Platform-specific management interface for the thread system
     * of the Java virtual machine.
     * <p>
     * This platform extension is only available to a thread
     * implementation that supports this extension.
     *
     * @author  Paul Hohensee
     * @since   6u25
     */
    ...
    public interface ThreadMXBean extends java.lang.management.ThreadMXBean {
```


* com.sun.management.UnixOperatingSystem  (extends sun.management.OperatingSystemImpl)

```java
    ((cite: jdk/src/solaris/classes/com/sun/management/UnixOperatingSystem.java))
    /**
     * Implementation class for the operating system.
     * Standard and committed hotspot-specific metrics if any.
     *
     * ManagementFactory.getOperatingSystemMXBean() returns an instance
     * of this class.
     */
    class UnixOperatingSystem
        extends    sun.management.OperatingSystemImpl
        implements UnixOperatingSystemMXBean {
```

* com.sun.management.OperatingSystem      (extends sun.management.OperatingSystemImpl)

```java
    ((cite: jdk/src/windows/classes/com/sun/management/OperatingSystem.java))
    /**
     * Implementation class for the operating system.
     * Standard and committed hotspot-specific metrics if any.
     *
     * ManagementFactory.getOperatingSystemMXBean() returns an instance
     * of this class.
     */
    class OperatingSystem
        extends    sun.management.OperatingSystemImpl
        implements OperatingSystemMXBean {
```




## Subcategories
* [Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： JMM Interface](noRM-0G7af.html)



