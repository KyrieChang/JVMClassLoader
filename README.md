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
   ![Aaron Swartz](https://github.com/KyrieChang/JVMClassLoader/blob/master/Image.png)
   **注意：父子类加载器不一定是继承关系，子类加载器不一定继承父类加载器！！！所有加载器有且只有一个父类加载器（根加载器除外）**
 ### 三、父委托机制
   * 父委托机制如下：
   > Class sampleClass = loader4.loaderClass("Sample");
   loader1 <- loader2 <- loader3 <- loader4
   首先loader4从自己的命名空间中查找Sample 类是否已经被加载，如果已经加载，就直接返回
   Sample 类的Class 对象的引用。如果没有被加载，那么4委托3去加载，3委托2去加载，2委托1去加载！！
   如果1能加载，那么就直接返回Calss 对象的引用，如果1不能加载，那么2就去加载、、、直到4去加载。
   如果4的所有父加载器，以及4也不能加载，那么将抛出一个ClassNotFoundException 异常
### 四、命名空间
  * 每个类加载器都有自己的命名空间，命名空间由该类加载器以及所有的父类加载器所构成，在同一个
    命名空间中不会出现类完整名字相同的类；在不同的命名空间中有可能会出现类的完整名称相同的类！！
  > * 如：loader1 <- loader2    是父子加载器    loader3 跟他们没关系,那么，如果loader2 加载一个A类，那么它的父类
  loader1 再去加载A类是不成功的。  但是loader3 可以再次把A类加载到内存中！！！因为它们在不同的命名空间中。
  
### 五、运行时包
  * 由同一个类加载器加载的属于相同的包的类组成了运行时包。决定两个类是不是同一个运行时包，不仅要看它们
  的包名是否相同，还要看定义类加载器是否相同。只有属于同一运行时包的类才能互相访问包可见
  （即默认访问级别）的类和类成员。
  * 作用：
  这样的限制能避免用户自定义的类冒充核心类库的包可见成员。假设用户自定义了一个类java.lang.Spy
  ,并且由用户自定义的类加载器加载，由于java.lang.Spy 和核心类库java.lang.* 由不同的加载器
  加载，它们属于不同的运行时包，所以java.lang.Spy 不能访问核心类库java.lang包中的包可见成员
  
### 六、自定义类加载器
  * 必须要重写ClassLoader类的findClass() 方法

  
### 结束，代码之后贴上

    
