# JVMClassLoader
JVMClassLoader

## Java虚拟机与程序的生命周期
   **1、执行了System.exit() 方法。**

   **2、程序正常执行结束。**

   **3、程序在执行过程中遇到了异常或者错误而导致程序终止。**

   **4、由于操作系统出现错误而导致Java虚拟机终止。**

## JVM 类加载顺序(加载，连接，初始化)
 ### 加载：查找并加载类的二进制数据到内存中
   **说明：类的加载是指将类的.class文件二进制数据读入内存中，将其放在运行时方法区内，然后在堆区创建一个java.lang.Class 对象，用来封装类在方法区内的数据结构。（java.lang.Class 对象是jvm帮我们创建的，例如：反射的时候可以用到）*
     > 重要：加载.class 文件的方式
     * 从本地磁盘加载（最简单的一种方式）
     * 通过网络下载.class 文件（URLClassLoader 常见）
     * 从zip,jar 等文件中加载.class 文件（常见）
     * 从专有的数据库中提取.class 文件(不常见)
     * 将java源文件动态编译为.class 文件（不常见）
     
   **每一个类的class对象是jvm 在加载类的时候就创建在内存中，使用的时候直接获取即可！！！如：Class clazz = Class.forName("java.lang.String");**
  ### 连接
   * 验证：确保被加载类的正确性（防止恶意的class文件）
   * 准备：为类的静态变量分配内存，并将其转化为默认值（int:0;boolean:false;引用类型：null）
   * 解析：把类中的符号引用转化为直接引用
     
  ### 初始化：为类的静态变量赋予正确的初始值
   > 例如
   <pre><code>public class Test{
       private static int nums = 0;  (准备阶段默认值为0，初始化之后正确赋值为0)
       private static Object obj = new Object(); (准备阶段默认值为null,初始化之后正确的赋值为object 的实例)
       private static boolean flag = true; (准备阶段默认值为false,初始化之后的正确赋值为true)
    }
   </code></pre>
