内存区域只jvm在执行java程序的内存区域
1. 方法区：存放类信息、常量、静态变量、即时编译器编译后的代码等。
2. 堆：存放对象实例，包括新创建的对象、数组等。
3. 栈：存放方法调用的局部变量、操作数、返回地址等。
4. 程序计数器：存放当前线程执行字节码的行号指示器。 
JVM在执行java程序时，会在堆、方法区、栈、程序计数器等内存区域分配内存，并在这些内存区域中存储和管理java程序运行时的数据。

堆和方法区是线程共享的，本地方法栈和程序计数器是线程私有的。

```
public class MemoryAllcation {
    // 静态变量, 分配在方法区
    static int staticInt;

    // 实例变量，分配在堆
    private int instanceInt;

    // 方法中的局部变量，分配在虚拟机栈的局部变量表中
    public void method() {
        int localInt = 0; // 局部变量，分配在虚拟机栈的局部变量表中
        System.out.println("localInt = " + localInt);

        Person person = new Person(); // 堆内存分配
        person.name = "Tom"; // 堆内存分配
        System.out.println("person.name = " + person.name);
    }

    public static void main(String[] args) {
        MemoryAllcation memoryAllcation = new MemoryAllcation();
    }
```

Java对象不都是分配在堆上吗？其实不然，有的对象是分配在栈上，比如局部变量、方法参数、返回值等。
逃逸分析用于确定对象是否应该分配在堆上还是栈上。如果没有发生逃逸，则分配在栈上，否则分配在堆上。

```
public class EscapeAnalysis {

    // 创建的对象没有外部方法引用
    // 也没有任何外部变量持有引用，不会逃逸到
    public void noEscape() {
        int a = 10; // a 不会逃逸到堆上
        int b = 20; // b 不会逃逸到堆上
        int c = a + b; // c 不会逃逸到堆上
        System.out.println("c = " + c);
    }

    // 创建的对象作为方法的返回值，会逃逸到堆上
    public Person escape() {
        Person person = new Person(); // person 逃逸到堆上
        person.name = "Tom"; // person 逃逸到堆上
        return person; // person 逃逸到堆上
    }
    
    public static void main(String[] args) {
        int a = 10; // a 不会逃逸到堆上
        int b = 20; // b 不会逃逸到堆上
        int c = a + b; // c 逃逸到堆上
}}

```
逃逸分析可以带来的优化
1. 减少堆内存的使用，减少垃圾回收的频率，提高程序的运行效率。
2. 减少方法调用的开销，减少方法调用的次数，提高程序的运行效率。
3. 减少内存碎片，提高程序的运行效率。


类加载机制
加载过程：
1. 加载：将class文件字节码读入内存，并为之创建一个Class对象。
2. 链接：验证、准备、解析。
    验证： 验证字节流信息符合当前虚拟机需求，防止篡改过的字节码危害JVM
    准备： 为类变量分配内存并设置默认初始值，静态变量在方法区中分配内存，并设置默认初始值。
    解析：将符号引用转换为直接引用，符号引用就是在编译期间生成的字符串，直接引用就是在运行期间解析出真实的内存地址。
3. 初始化：执行类变量的初始化。

```
step1: 创建一个java类
public class Helloworld {
    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}

step2: 编译java类
javac Helloworld.java

step3: 加载class文件 (自定义类加载器)
import java.io.*;
public class CustomClassLoader extends ClassLoader {

    public CustomClassLoader() {
        super();
    }

    public Class loadClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        return defineClass(name, classData, 0, classData.length);
    }

    private byte[] loadClassData(String name) {
        String classFileName = name.replace('.', '/') + ".class";
        try {
            InputStream inputStream = this.getClass().getResourceAsStream(classFileName);
            if (inputStream == null) {
                return null;
            }
            ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
            int bufferSize = 1024;
            byte[] buffer = new byte[bufferSize];
            int length;
            while ((length = inputStream.read(buffer)) != -1) {
                outputStream.write(buffer, 0, length);
            }
            return outputStream.toByteArray();
        } catch (IOException e) {            
            e.printStackTrace();
            return null;
        }
    }
}

step4: 自定义类加载器加载类
CustomClassLoader customClassLoader = new CustomClassLoader();
Class helloworldClass = customClassLoader.loadClass("Helloworld");

step5: 创建对象并调用方法
Object obj = helloworldClass.newInstance();
Method method = helloworldClass.getMethod("main", String[].class);
method.invoke(obj, new Object[] {args});
```

类加载器
1. 启动类加载器：BootstrapClassLoader，负责加载java核心类库，如java.lang.*。 java_home/lib
2. 扩展类加载器：ExtensionClassLoader，负责加载java扩展类库，如javax.*。 loaded from java_home/lib/ext
3. 应用程序类加载器：AppClassLoader，负责加载用户自定义类。

双亲委派模型
当一个类需要被加载时，ClassLoader会先委托父类加载器加载，如果父类加载器无法加载，才由自己加载。
优点：避免类的重复加载，保证类加载的安全性。其次防止恶意覆盖java核心类库。

三次大型破坏双亲委派模型