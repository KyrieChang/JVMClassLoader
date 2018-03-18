# JVMClassLoader
JVMClassLoader

## Java虚拟机与程序的生命周期
   **1、执行了System.exit() 方法。**

   **2、程序正常执行结束。**

   **3、程序在执行过程中遇到了异常或者错误而导致程序终止。**

   **4、由于操作系统出现错误而导致Java虚拟机终止。**

## JVM 类加载顺序(加载，连接，初始化)
 ### 加载：查找并加载类的二进制数据到内存中
   *说明：类的加载是指将类的.class文件二进制数据读入内存中，将其放在运行时方法区内，然后在堆区创建一个java.lang.Class 对象，用来封装类在方法区内的数据结构。（java.lang.Class 对象是jvm帮我们创建的，例如：反射的时候可以用到）*
   > 重要：加载.class 文件的方式
   > * 从本地磁盘加载（最简单的一种方式）
   > * 通过网络下载.class 文件（URLClassLoader 常见）
   > * 从zip,jar 等文件中加载.class 文件（常见）
   > * 从专有的数据库中提取.class 文件(不常见)
   > * 将java源文件动态编译为.class 文件（不常见）
     
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
  ### 什么情况初始化？
   * 首先说一个知识点，Java 程序对类的使用分两种
   * 主动使用（初始化）
   * 被动使用
   > 主动使用的场景如下：(必须首次主动使用才会初始化)
   > * 创建类的实例
   > * 访问某个类或者接口的静态变量，或者对该静态变量赋值（A.nums=0 等等）
   > * 调用类的静态方法(A.method())
   > * 反射(如 Class.forName(com.kyrie.peng.Test))
   > * 初始化一个类的子类
   > * Java虚拟机启动时被标明为启动类的类(Java Test,比如这个类里面存在main方法)
   
## 类加载器（父委托机制）
 ### 一、虚拟机自带的加载器
   * 根类加载器(Bootstrap），使用C++编写，java程序员无法在程序中获取
   > 该类没有父类加载器，它是负责加载虚拟机的核心类库，比如java.lang.*等
   <pre><code>
   Class clazz = Class.forName("java.lang.String");
   System.out.println(clazz.getClassLoader()); // 打印结果为 null，这个是根类加载器加载的，是得不到这个加载器的
   </code></pre>
   * 扩展类加载器（Extension），使用java代码实现
   > 该类的父加载器为根类加载器，它是从java.ext.dir 系统属性所指定的目录中加载类库，或者从jdk的安装目录的jre\lib\ext
   子目录（拓展目录）下加载类库，如果用户创建的JAR 文件放在这个目录下，也会自动由扩展类加载器加载。拓展类加载器是纯java类
   ，是java.lang.ClassLoader 的子类。
   * 系统类加载器（应用类加载器System）,使用java 代码实现
   > 它的父加载器是拓展类加载器。它从环境变量classpath或者系统属性java.class.path 所指定的目录中加载类，它是用户自定义的类加载
   器的父加载器，系统类加载器是纯java 类，是java.lang.Classloader类的子类.
 ### 二、自定义类加载器
   * java.lang.ClassLoader 的子类
   > * java 提供了抽象类java.lang.ClassLoader,所有的自定义的类加载器应该继承Classloader
   > * 1、当生成了一个自定义的类加载器实例时，如果没有指定它的父加载器，那么系统类加载器就成为它的父类加载器（默认构造方法）
   > * 2、当生成了一个自定义加载器实例时，如果指定了它的父加载器，那么指定的加载器就成为新new出来的加载器的父加载器（有参数的构造方法）
   > * 关系图如下：
 
   
