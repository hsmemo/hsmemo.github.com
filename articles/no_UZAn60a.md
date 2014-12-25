---
layout: default
title: Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean オブジェクトの取得処理 (java.lang.management.ManagementFactory.get*MXBean() の処理)
---
[Up](no2114S_x.html) [Top](../index.html)

#### Serviceability 機能 ： HotSpot Monitoring and Management Interface (JMM) ： 各種 Platform MXBean オブジェクトの取得処理 (java.lang.management.ManagementFactory.get*MXBean() の処理)

--- 
## 概要(Summary)
各種 Platform MXBean オブジェクトの取得処理は, java.lang.management.ManagementFactory.get${対応するMXBeanクラス名}MXBean() メソッドで行われる.

内部的には sun.management.ManagementFactoryHelper というクラスが使用されており, 
これにより適切な Platform MXBean 実装の取得が行われる.

## 処理の流れ (概要)(Execution Flows : Summary)
### java.lang.management.ManagementFactory.getOperatingSystemMXBean() の処理
```
java.lang.management.ManagementFactory.getOperatingSystemMXBean()
 -> sun.management.ManagementFactoryHelper.getOperatingSystemMXBean()
    -> com.sun.management.OSMBeanFactory.getOperatingSystemMXBean()
       (このメソッドは OS 毎に実装が用意されている)
       * Solaris の場合 (Linux も兼用?? #TODO):
         -> com.sun.management.UnixOperatingSystem のインスタンスが返される
       * Windows の場合:
         -> com.sun.management.OperatingSystem のインスタンスが返される
```

### java.lang.management.ManagementFactory.getRuntimeMXBean() の処理
```
java.lang.management.ManagementFactory.getRuntimeMXBean()
 -> sun.management.ManagementFactoryHelper.getRuntimeMXBean()
    -> sun.management.RuntimeImpl のインスタンスが返される.
```

### java.lang.management.ManagementFactory.getCompilationMXBean() の処理
```
java.lang.management.ManagementFactory.getCompilationMXBean()
 -> sun.management.ManagementFactoryHelper.getCompilationMXBean()
    -> sun.management.CompilationImpl のインスタンスが返される.
```

### java.lang.management.ManagementFactory.getClassLoadingMXBean() の処理
```
java.lang.management.ManagementFactory.getClassLoadingMXBean()
 -> sun.management.ManagementFactoryHelper.getClassLoadingMXBean()
    -> sun.management.ClassLoadingImpl のインスタンスが返される.
```

### java.lang.management.ManagementFactory.getThreadMXBean() の処理
```
java.lang.management.ManagementFactory.getThreadMXBean()
 -> sun.management.ManagementFactoryHelper.getThreadMXBean()
    -> sun.management.ThreadImpl のインスタンスが返される.
```

### java.lang.management.ManagementFactory.getMemoryMXBean() の処理
```
java.lang.management.ManagementFactory.getMemoryMXBean()
 -> sun.management.ManagementFactoryHelper.getMemoryMXBean()
    -> sun.management.MemoryImpl のインスタンスが返される.
```

### java.lang.management.ManagementFactory.getGarbageCollectorMXBeans() の処理
```
java.lang.management.ManagementFactory.getGarbageCollectorMXBeans()
 -> sun.management.ManagementFactoryHelper.getGarbageCollectorMXBeans()
    -> sun.management.MemoryImpl.getMemoryManagers()
       -> sun.management.MemoryImpl.getMemoryManagers0()
          -> Java_sun_management_MemoryImpl_getMemoryManagers0()
             -> jmm_GetMemoryManagers()
                -> MemoryManager::get_memory_manager_instance()
                   -> sun.management.ManagementFactory.createGarbageCollector()
                      -> sun.management.GarbageCollectorImpl のインスタンスが返される
```

### java.lang.management.ManagementFactory.getMemoryManagerMXBeans() の処理
```
java.lang.management.ManagementFactory.getMemoryManagerMXBeans()
 -> sun.management.ManagementFactoryHelper.getMemoryManagerMXBeans()
    -> sun.management.MemoryImpl.getMemoryManagers()
       -> sun.management.MemoryImpl.getMemoryManagers0()
          -> Java_sun_management_MemoryImpl_getMemoryManagers0()
             -> jmm_GetMemoryManagers()
                -> MemoryManager::get_memory_manager_instance()
                   -> sun.management.ManagementFactory.createMemoryManager()
                      -> sun.management.MemoryManagerImpl のインスタンスが返される
```

### java.lang.management.ManagementFactory.getMemoryPoolMXBeans() の処理
```
java.lang.management.ManagementFactory.getMemoryPoolMXBeans()
 -> sun.management.ManagementFactoryHelper.getMemoryPoolMXBeans()
    -> sun.management.MemoryImpl.getMemoryPools()
       -> sun.management.MemoryImpl.getMemoryPools0()
          -> Java_sun_management_MemoryImpl_getMemoryPools0()
             -> jmm_GetMemoryPools()
                -> MemoryPool::get_memory_pool_instance()
                   -> sun.management.ManagementFactory.createMemoryPool()
                      -> sun.management.MemoryPoolImpl のインスタンスが返される
```

## 備考(Notes)
なお, HotSpot 独自の MBean については以下のメソッドで取得できる.


```java
    ((cite: jdk/src/share/classes/sun/management/ManagementFactoryHelper.java))
        public static synchronized HotSpotDiagnosticMXBean getDiagnosticMXBean() {
    ...
    
        /**
         * This method is for testing only.
         */
        public static synchronized HotspotRuntimeMBean getHotspotRuntimeMBean() {
    ...
    
        /**
         * This method is for testing only.
         */
        public static synchronized HotspotClassLoadingMBean getHotspotClassLoadingMBean() {
    ...
    
        /**
         * This method is for testing only.
         */
        public static synchronized HotspotThreadMBean getHotspotThreadMBean() {
    ...
    
        /**
         * This method is for testing only.
         */
        public static synchronized HotspotMemoryMBean getHotspotMemoryMBean() {
    ...
    
        /**
         * This method is for testing only.
         */
        public static synchronized HotspotCompilationMBean getHotspotCompilationMBean() {
    ...
```


## 処理の流れ (詳細)(Execution Flows : Details)
### java.lang.management.ManagementFactory の各種メソッド

```java
    ((cite: jdk/src/share/classes/java/lang/management/ManagementFactory.java))
    package java.lang.management;
    ...
    public class ManagementFactory {
    ...
        /**
         * Returns the managed bean for the class loading system of
         * the Java virtual machine.
         *
         * @return a {@link ClassLoadingMXBean} object for
         * the Java virtual machine.
         */
        public static ClassLoadingMXBean getClassLoadingMXBean() {
            return ManagementFactoryHelper.getClassLoadingMXBean();
        }
    
        /**
         * Returns the managed bean for the memory system of
         * the Java virtual machine.
         *
         * @return a {@link MemoryMXBean} object for the Java virtual machine.
         */
        public static MemoryMXBean getMemoryMXBean() {
            return ManagementFactoryHelper.getMemoryMXBean();
        }
    
        /**
         * Returns the managed bean for the thread system of
         * the Java virtual machine.
         *
         * @return a {@link ThreadMXBean} object for the Java virtual machine.
         */
        public static ThreadMXBean getThreadMXBean() {
            return ManagementFactoryHelper.getThreadMXBean();
        }
    
        /**
         * Returns the managed bean for the runtime system of
         * the Java virtual machine.
         *
         * @return a {@link RuntimeMXBean} object for the Java virtual machine.
    
         */
        public static RuntimeMXBean getRuntimeMXBean() {
            return ManagementFactoryHelper.getRuntimeMXBean();
        }
    
        /**
         * Returns the managed bean for the compilation system of
         * the Java virtual machine.  This method returns <tt>null</tt>
         * if the Java virtual machine has no compilation system.
         *
         * @return a {@link CompilationMXBean} object for the Java virtual
         *   machine or <tt>null</tt> if the Java virtual machine has
         *   no compilation system.
         */
        public static CompilationMXBean getCompilationMXBean() {
            return ManagementFactoryHelper.getCompilationMXBean();
        }
    
        /**
         * Returns the managed bean for the operating system on which
         * the Java virtual machine is running.
         *
         * @return an {@link OperatingSystemMXBean} object for
         * the Java virtual machine.
         */
        public static OperatingSystemMXBean getOperatingSystemMXBean() {
            return ManagementFactoryHelper.getOperatingSystemMXBean();
        }
    
        /**
         * Returns a list of {@link MemoryPoolMXBean} objects in the
         * Java virtual machine.
         * The Java virtual machine can have one or more memory pools.
         * It may add or remove memory pools during execution.
         *
         * @return a list of <tt>MemoryPoolMXBean</tt> objects.
         *
         */
        public static List<MemoryPoolMXBean> getMemoryPoolMXBeans() {
            return ManagementFactoryHelper.getMemoryPoolMXBeans();
        }
    
        /**
         * Returns a list of {@link MemoryManagerMXBean} objects
         * in the Java virtual machine.
         * The Java virtual machine can have one or more memory managers.
         * It may add or remove memory managers during execution.
         *
         * @return a list of <tt>MemoryManagerMXBean</tt> objects.
         *
         */
        public static List<MemoryManagerMXBean> getMemoryManagerMXBeans() {
            return ManagementFactoryHelper.getMemoryManagerMXBeans();
        }
    
    
        /**
         * Returns a list of {@link GarbageCollectorMXBean} objects
         * in the Java virtual machine.
         * The Java virtual machine may have one or more
         * <tt>GarbageCollectorMXBean</tt> objects.
         * It may add or remove <tt>GarbageCollectorMXBean</tt>
         * during execution.
         *
         * @return a list of <tt>GarbageCollectorMXBean</tt> objects.
         *
         */
        public static List<GarbageCollectorMXBean> getGarbageCollectorMXBeans() {
            return ManagementFactoryHelper.getGarbageCollectorMXBeans();
        }
```


### sun.management.ManagementFactoryHelper の各種メソッド

```java
    ((cite: jdk/src/share/classes/sun/management/ManagementFactoryHelper.java))
    package sun.management;
    ...
    /**
     * ManagementFactoryHelper provides static factory methods to create
     * instances of the management interface.
     */
    public class ManagementFactoryHelper {
    ...
        public static synchronized ClassLoadingMXBean getClassLoadingMXBean() {
            if (classMBean == null) {
                classMBean = new ClassLoadingImpl(jvm);
            }
            return classMBean;
        }
    
        public static synchronized MemoryMXBean getMemoryMXBean() {
            if (memoryMBean == null) {
                memoryMBean = new MemoryImpl(jvm);
            }
            return memoryMBean;
        }
    
        public static synchronized ThreadMXBean getThreadMXBean() {
            if (threadMBean == null) {
                threadMBean = new ThreadImpl(jvm);
            }
            return threadMBean;
        }
    
        public static synchronized RuntimeMXBean getRuntimeMXBean() {
            if (runtimeMBean == null) {
                runtimeMBean = new RuntimeImpl(jvm);
            }
            return runtimeMBean;
        }
    
        public static synchronized CompilationMXBean getCompilationMXBean() {
            if (compileMBean == null && jvm.getCompilerName() != null) {
                compileMBean = new CompilationImpl(jvm);
            }
            return compileMBean;
        }
    
        public static synchronized OperatingSystemMXBean getOperatingSystemMXBean() {
            if (osMBean == null) {
                osMBean = (OperatingSystemImpl)
                              OSMBeanFactory.getOperatingSystemMXBean(jvm);
            }
            return osMBean;
        }
    
        public static List<MemoryPoolMXBean> getMemoryPoolMXBeans() {
            MemoryPoolMXBean[] pools = MemoryImpl.getMemoryPools();
            List<MemoryPoolMXBean> list = new ArrayList<MemoryPoolMXBean>(pools.length);
            for (MemoryPoolMXBean p : pools) {
                list.add(p);
            }
            return list;
        }
    
        public static List<MemoryManagerMXBean> getMemoryManagerMXBeans() {
            MemoryManagerMXBean[]  mgrs = MemoryImpl.getMemoryManagers();
            List<MemoryManagerMXBean> result = new ArrayList<MemoryManagerMXBean>(mgrs.length);
            for (MemoryManagerMXBean m : mgrs) {
                result.add(m);
            }
            return result;
        }
    
        public static List<GarbageCollectorMXBean> getGarbageCollectorMXBeans() {
            MemoryManagerMXBean[]  mgrs = MemoryImpl.getMemoryManagers();
            List<GarbageCollectorMXBean> result = new ArrayList<GarbageCollectorMXBean>(mgrs.length);
            for (MemoryManagerMXBean m : mgrs) {
                if (GarbageCollectorMXBean.class.isInstance(m)) {
                     result.add(GarbageCollectorMXBean.class.cast(m));
                }
            }
            return result;
        }
```


### com.sun.management.OSMBeanFactory.getOperatingSystemMXBean() (Solaris の場合)
See: [here](no2114vHA.html) for details
### com.sun.management.OSMBeanFactory.getOperatingSystemMXBean() (Windows の場合)
See: [here](no21148RG.html) for details

### sun.management.MemoryImpl.getMemoryManagers()
See: [here](no2114ekB.html) for details
### Java_sun_management_MemoryImpl_getMemoryManagers0()
See: [here](no2114SNa.html) for details
### jmm_GetMemoryManagers()
See: [here](no2114shm.html) for details
### MemoryManager::get_memory_manager_instance()
See: [here](no21148KS.html) for details
### sun.management.ManagementFactory.createMemoryManager()
See: [here](no2114Wfe.html) for details
### sun.management.ManagementFactory.createGarbageCollector()
See: [here](no2114jpk.html) for details

### sun.management.MemoryImpl.getMemoryPools()
See: [here](no2114ruH.html) for details
### Java_sun_management_MemoryImpl_getMemoryPools0()
See: [here](no2114FDU.html) for details
### jmm_GetMemoryPools()
See: [here](no2114fXg.html) for details
### MemoryPool::get_memory_pool_instance()
See: [here](no2114wzq.html) for details
### sun.management.ManagementFactory.createMemoryPool()
See: [here](no211499w.html) for details






