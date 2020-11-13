## 2. Java运行区域与内存溢出异常

### 2.1 运行时数据区

<div style="display: flex;text-align:center;">
    <div style="flex:1;">
        jdk1.8之前
        <img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E8%BF%90%E8%A1%8C%E6%97%B6%E5%8C%BA%E5%9F%9F1.png" />
    </div>
    <div style="flex:1;">
        jdk1.8
        <img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E8%BF%90%E8%A1%8C%E6%97%B6%E5%8C%BA%E5%9F%9F2.png" />
    </div>
</div>

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/jvm%E8%BF%90%E8%A1%8C%E6%97%B6%E6%95%B0%E6%8D%AE%E5%8C%BA-%E5%85%83%E7%A9%BA%E9%97%B4%E7%89%88.webp" style="zoom:70%;" />

#### 线程隔离的数据区(私有)

1. **程序计数器**

    程序计数器可以看作是当前线程所执行的字节码的行号指示器。为了线程切换后能恢复到正确的执行位置， 每条线程都需要有一个独立的程序计数器， 各条线程之间计数器互不影响， 独立存储。

2. **Java虚拟机栈**

    私有，生命周期与线程生命周期相同。

    **每一个方法被调用直至执行完毕的过程， 就对应着一个栈帧在虚拟机栈中从入栈到出栈的过程**。

    > 栈帧用于存储 ==局部变量表、操作数栈、常量池引用==等信息。局部变量表存放了编译期可知的各种Java虚拟机基本数据类型，对象引用，returnAddress 类型。

    可以通过 **-Xss** 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小，在 JDK 1.4 中默认为 256K，而在 JDK 1.5+ 默认为 1M。  

    该区域可能抛出的异常：

    - 当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常。
    - 栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。

3. **本地方法栈**

    与虚拟机栈类似，虚拟机栈为虚拟机执行Java方法（也就是字节码） 服务， 而本地方法栈则是为虚拟机使用到的本地（Native）方法服务。

    本地⽅法被执⾏的时候，在本地⽅法栈也会创建⼀个栈帧，⽤于存放该本地⽅法的局部变量表、操作数栈、动态链接、出⼝信息。

    ⽅法执⾏完毕后相应的栈帧也会出栈并释放内存空间，也会出现 StackOverFlowError 和 OutOfMemoryError 两种异常 。

#### 所有线程共享的数据区(公有)

1. **Java堆**

    Java堆是被所有线程共享的一块内存区域， 在虚拟机启动时创建。 **此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数组都应当在堆上分配**。

    Java堆是垃圾收集器管理的内存区域， 因此它也被称作“GC堆”。

    堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。

    可以通过  **-Xms**  和 **-Xmx**  这两个虚拟机参数来指定一个程序的堆内存大小，第一个参数设置初始值，第二个参数设置最大值。

    ```java
    java -Xms1M -Xmx2M HackTheJava
    ```

2. **方法区**

    用于存储已被虚拟机加载的==类型信息、 常量、 静态变量、 即时编译器编译后的代码缓存==等数据。

    和堆一样不需要连续的内存，并且可以动态扩展，动态扩展失败一样会抛出 OutOfMemoryError 异常。

    **JDK1.8之前HotSpot虚拟机使用永久代来实现方法区，到了1.8完全废弃永久代的概念，改用在本地内存中实现的元空间（Metaspace）来代替。**

    **常用参数**：

    - 1.8之前

        ```java
        -XX:PermSize=N //⽅法区(永久代)初始⼤⼩
        -XX:MaxPermSize=N //⽅法区(永久代)最⼤⼤⼩,超过这个值将会抛出 OutOfMemoryError 异常
        ```

    - 1.8及其之后

        ```java
        -XX:MetaspaceSize=N //设置 Metaspace 的初始（和最⼩⼤⼩）
        -XX:MaxMetaspaceSize=N //设置 Metaspace 的最⼤⼤⼩ 
        ```

3. **运行时常量池**

    运行时常量池是方法区的一部分，Class 文件中的常量池（编译器生成的字面量和符号引用）会在类加载后被放入这个区域。

#### 直接内存

直接内存**不属于虚拟机运行时数据区**，但是这部分内存也被频繁地使用， 而且也可能导 OutOfMemoryError 异常出现。



### 2.2. HotSpot 虚拟机对象

#### Java 对象的创建

<center>
    <img src="https://gitee.com/coldsun233/NotePic/raw/master/img/java%E5%AF%B9%E8%B1%A1%E5%88%9B%E5%BB%BA%E8%BF%87%E7%A8%8B.png"/>
</center>



1. **类加载检查：**当 Java 虚拟机遇到一条字节码 new 指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

2. **分配内存：**在类加载检查通过后，接下来虚拟机将为新生对象分配内存。对象所需内存的大小在类加载完成后便可完全确定。

    **内存分配方式：**

    - **如果 Java 堆中内存是绝对规整的**，所有被使用过的内存都被放在一边，空闲的内存被放在另一边，中间放着一个指针作为分界点的指示器，那所分配内存就仅仅是把那个指针向空闲空间方向挪动一段与对象大小相等的距离，这种分配方式称为“**指针碰撞**”。

    - **如果 Java 堆中的内存并不是规整的**， 虚拟机就必须维护一个列表，记录上哪些内存块是可用的，在分配的时候从列表中找到一块足够大的空间划分给对象实例，并更新列表上的记录，这种分配方式称为“**空闲列表**”。

        > Java 堆内存是否规整，取决于 GC 收集器的算法是**"标记-清除"（不规整）**，还是**"标记-整理"（也称作"标记-压缩"，规整）**，其次，**使用复制算法的内存也是规整的**。

    **解决内存分配的并发问题**

    - 方法一：对分配内存空间的动作进行同步处理，虚拟机是采用 **CAS + 失败重试**的方式保证更新操作的原子性 。
    - 方法二：把内存分配的动作按照线程划分在不同的空间之中进行， 即每个线程在Java堆中预先分配一小块内存， 称为**本地线程分配缓冲**（Thread Local AllocationBuffer， TLAB） ， 哪个线程要分配内存， 就在哪个线程的本地缓冲区中分配， 只有本地缓冲区用完了， 分配新的缓存区时才需要同步锁定。

3. **初始化零值：**内存分配完成后，虚拟机需要将分配到的内存空间都初始化为零值（不包括对象头），这⼀步操作保证了对象的实例字段在 Java 代码中可以不赋初始值就直接使⽤。

4. **设置对象头：**初始化零值完成之后， 虚拟机要对对象进⾏必要的设置，例如这个对象是那个类的实例、如何才能找到类的元数据信息、对象的哈希吗、对象的 GC 分代年龄等信息。  

    -----

5. **执行init方法：**1~4步执行完后，对于虚拟机来说对象创建完成，但Java对象的创建还需要继续执行构造函数，即 Class 类的\<init>()方法，按照程序员的意愿对对象进行初始化， 这样一个真正可用的对象才算完全被构造出来。

#### 对象的内存布局

​	在 HotSpot 虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：对象头（header）、实例数据（Instance Data）、对齐填充（Padding）。

- **对象头**：HotSpot 虚拟机对象的对象头部分包括两类信息 。第一类是用于存储对象自身的运行时数据 ，被称为**“Mark word”**；第二类是类型指针， 即对象指向它的类型元数据的指针  ，如果对象是一个 Java 数组， 那在对象头中还必须有一块用于记录数组长度的数据。
- **实例数据**：是对象真正存储的有效信息， 即我们在程序代码里面所定义的各种类型的字段内容， 无论是从父类继承下来的， 还是在子类中定义的字段都必须记录起来。

 - **对齐填充**：HotSpot 虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍， 换句话说就是任何对象的大小都必须是8字节的整数倍，**如果对象实例数据部分没有对齐的话， 就需要通过对齐填充来补全**。

#### 对象的访问定位

​	主流的访问方式主要有使用**句柄**和**直接指针**两种：

1. 句柄：使用句柄访问， Java堆中将可能会划分出一块内存来作为句柄池， **reference 中存储的就是对象的句柄地址**， 而句柄中包含了对象实例数据与类型数据各自具体的地址信息。

    <img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E5%8F%A5%E6%9F%84%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1.png" style="zoom:90%;" />

    **优点**：reference 中存储的是稳定句柄地址， 在对象被移动（垃圾收集时移动对象是非常普遍的行为）时只会改变句柄中的实例数据指针， 而 reference 本身不需要被修改。

2. 直接指针：使用直接指针访问，Java堆中对象的内存布局就必须考虑如何放置访问类型数据的相关信息，**reference中存储的直接就是对象地址**，如果只是访问对象本身的话， 就不需要多一次间接访问的开销。

    <img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E7%9B%B4%E6%8E%A5%E6%8C%87%E9%92%88%E8%AE%BF%E9%97%AE%E5%AF%B9%E8%B1%A1.png" style="zoom:90%;" />

    **优点**：速度更快， 因为它节省了一次指针定位的时间开销。

<!-- 分页-->

<div style="page-break-after: always;"></div>

<!--分页-->

## 3. 垃圾收集器与内存分配策略

### 3.1 判断对象是否死亡

#### 引用计数法

​	在对象中添加一个引用计数器，每当有一个地方引用它时，计数器值就加一，当引用失效时，计数器值就减一，任何时刻计数器为零的对象就是不可能再被使用的。

**不足**：单纯的引用计数很难解决对象之间相互循环引用的问题。当两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。

```java
public class Test {
    public Object instance = null;
    
    public static void main(String[] args) {
        Test a = new Test();
        Test b = new Test();
        a.instance = b;
        b.instance = a;
        a = null;
        b = null;
        ...
    }
}
```

​	在上述代码中，a 与 b 引用的对象实例互相持有了对象的引用，因此当我们把对 a 对象与 b 对象的引用去除之后，由于两个对象还存在互相之间的引用，导致两个 Test 对象无法被回收。

#### 可达性分析法

算法的基本思路就是通过一系列称为“GC Roots”的根对象作为起始节点集，从这些节点开始，根据引用关系向下搜索，搜索过程所走过的路径称为“引用链”（Reference Chain），如果某个对象到GC Roots间没有任何引用链相连，或者用图论的话来说就是从GC Roots到这个对象不可达时，则证明此对象是不可能再被使用的。

**固定可作为GC Roots的对象**：

1.**在虚拟机栈（栈帧中的本地变量表） 中引用的对象**

```java
public class StackLocalParameter {
    public StackLocalParameter(String name){}
}

public static void testGC(){
    StackLocalParameter s = new StackLocalParameter("localParameter");
    s = null;
}
```

此时的 s，即为 GC Root，当s置空时，localParameter 对象也断掉了与 GC Root 的引用链，将被回收。

2.**在方法区中类静态属性引用的对象**

```java
public class MethodAreaStaicProperties {
    public static MethodAreaStaicProperties m;
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("properties");
    s.m = new MethodAreaStaicProperties("parameter");
    s = null;
}
```

s 为 GC Root，s 置为 null，经过 GC 后，s 所指向的 properties 对象由于无法与 GC Root 建立关系被回收。

m 作为类的静态属性，也属于 GC Root，parameter 对象依然与 GC root 建立着连接，所以此时 parameter 对象并不会被回收。

3.**在方法区中常量引用的对象**

```java
public class MethodAreaStaicProperties {
    public static final MethodAreaStaicProperties m = MethodAreaStaicProperties("final");
    public MethodAreaStaicProperties(String name){}
}

public static void testGC(){
    MethodAreaStaicProperties s = new MethodAreaStaicProperties("staticProperties");
    s = null;
}
```

m 即为方法区中的常量引用，也为 GC Root，s 置为 null 后，final 对象也不会因没有与 GC Root 建立联系而被回收。

4.**本地方法栈中 JNI（即一般说的 Native 方法）引用的对象**

#### 引用

​	在 JDK1.2 版之前，Java 里面的引用是很传统的定义，如果reference类型的数据中存储的数值代表的是另外一块内存的起始地址， 就称该reference数据是代表某块内存、 某个对象的引用。

​	在 JDK1.2 版之后，Java 对引用的概念进行了扩充，将引用分为强引用（Strongly Reference）、软引用（Soft Reference） 、 弱引用（Weak Reference） 和虚引用（Phantom Reference） 4种， 这4种引用强度依次逐渐减弱。

- **强引用**

    指在程序代码之中普遍存在的引用赋值， 即类似“Object obj = new Object()”这种引用关系。

    无论任何情况下， 只要强引用关系还存在， 垃圾收集器就永远不会回收掉被引用的对象。

- **软引用**

    用于描述还有用但非必须的对象。

    只被软引用关联着的对象， 在系统将要发生内存溢出异常前，会把这些对象列进回收范围之中进行第二次回收，如果这次回收还没有足够的内存，才会抛出内存溢出异常。

- **弱引用**

    用来描述那些非必须对象，但是它的强度比软引用更弱一些。 

    被弱引用关联的对象只能生存到下一次垃圾收集发生为止。当垃圾收集器开始工作，无论当前内存是否足够，都会回收掉只被弱引用关联的对象。

- **虚引用**

    最弱的一种引用关系，为一个对象设置虚引用关联的唯一目的只是为了能在这个对象被收集器回收时收到一个系统通知。   

#### finalize()方法

​	即使在可达性分析算法中判定为不可达的对象， 也不是“非死不可”的， 这时候它们暂时还处于“缓刑”阶段。

​	finalize()方法是对象逃脱死亡命运的最后一次机会，对象要在finalize()中成功拯救自己只需要重新与引用链上的任何一个对象建立关联即可，如果对象这时候还没有逃脱， 那基本上它就真的要被回收了。

​	任何一个对象的finalize()方法都**只会被系统自动调用一次**， 如果对象面临下一次回收， 它的finalize()方法不会被再次执行，因此只能自救一次。

#### 回收方法区

​	方法区的垃圾收集主要回收两部分内容： 废弃的常量和不再使用的类型。

##### 如何判断一个常量是废弃常量

​	假如在常量池中存在一个字符串 "Java"，但是没有任何字符串对象引用该常量，且虚拟机中也没有其他地方引用这个常量，那它就是一个废弃常量。

##### 如何判定一个类型是否是“不再被使用的类”

​	需要同时满足三个条件：

- 该类所有的实例都已经被回收，此时堆中不存在该类及其任何派生子类的实例。
- 加载该类的类加载器已经被回收。
- 该类对应的 java.lang.Class 对象没有在任何地方被引用， 无法在任何地方通过反射访问该类的方法。

### 3.2 垃圾收集算法

#### 标记-清除算法

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E6%A0%87%E8%AE%B0%E6%B8%85%E9%99%A4%E7%AE%97%E6%B3%95.png" style="zoom:60%;" />

​	算法分为“标记”和“清除”两个阶段：**首先标记出所有需要回收的对象，在标记完成后，统一回收掉所有被标记的对象**，也可以反过来，标记存活的对象，统一回收所有未被标记的对象。   

**缺点**：

- 标记和清除过程**效率都不高**。

- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

#### 复制算法

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95.png" style="zoom:60%;" />

​	将可用内存按容量划分为大小相等的两块， 每次只使用其中的一块。 当这一块的内存用完了，就将还存活着的对象复制到另外一块上面， 然后再把已使用过的内存空间一次清理掉。

**优点**：保证了内存的连续可用，逻辑清晰，运行高效。

**缺点**：空间浪费太多。

#### 标记-整理算法

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E6%A0%87%E8%AE%B0%E6%95%B4%E7%90%86%E7%AE%97%E6%B3%95.png" style="zoom:60%;" />

​	标记过程仍然与标记--清除算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，再清理掉端边界以外的内存区域。

**优点**：不会产生内存碎片。

**缺点**：需要移动大量对象，处理效率比较低。

#### 分代收集

​	根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

​	在**新⽣代中，每次收集都会有⼤量对象死去，所以可以选择复制算法**，只需要付出少量对象的复制成本就可以完成每次垃圾收集。⽽**⽼年代的对象存活⼏率是⽐较⾼的，⽽且没有额外的空间对它进⾏分配担保，所以我们必须选择“标记-清除”或“标记 -- 整理”算法进⾏垃圾收集**。

### 3.3 堆的内存模型

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E5%A0%86%E7%9A%84%E5%88%86%E4%BB%A3%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B.png" style="zoom:60%;" />

​	Java 堆主要分为2个区域-年轻代与老年代，年轻代又分 Eden 区和 Survivor 区，Survivor 区又分 From 和 To 2个区。

#### Eden区

​	大多数情况下，对象会在新生代 Eden 区中进行分配，当 Eden 区没有足够空间进行分配时，虚拟机会发起一次 Minor GC。

​	通过 Minor GC 之后，Eden 会被清空，Eden 区中绝大部分对象会被回收，而那些无需回收的存活对象，将会进到 Survivor 的 From 区（若 From 区不够，无法安置的对象会进入 Old 区【内存担保机制】）。

#### Survivor区

​	Survivor 的存在意义就是**减少被送到老年代的对象，进而减少 Major GC 的发生**。Survivor 的预筛选保证，只有经历16次 Minor GC 还能在新生代中存活的对象，才会被送到老年代。

> **为什么需要两个Survivor？**
>
> ​	设置两个 Survivor 区最大的好处就是解决内存碎片化。新生代一般使用复制算法进行 GC，这就需要一个空闲的空间。Survivor 有2个区域，永远有一个是空闲的，假设第一次 Minor GC时，将 Eden 区和 From 区中的存活对象复制到 To 区域。第二次 Minor GC 时，From 与 To 职责兑换，这时候会将 Eden 区和 To 区中的存活对象再复制到 From 区域，以此反复。

#### Old区

​	只有在 Major GC 的时候才会进行清理，每次 GC 都会触发“Stop-The-World”。内存越大，STW 的时间也越长，所以内存也不仅仅是越大就越好。

​	由于复制算法在对象存活率较高的老年代会进行很多次的复制操作，效率很低，所以**老年代采用的是标记 -- 整理算法**。

### 3.4 内存分配与回收策略

1. **对象优先在 Eden 分配**

    大多数情况下，对象在新生代 Eden 区中分配。当 Eden 区没有足够空间进行分配时，虚拟机将发起一次 Minor GC。

2. **大对象直接进入老年代**

    **大对象就是指需要大量连续内存空间的Java对象**，最典型的大对象便是那种很长的字符串，或者元素数量很庞大的数组。

    **大对象直接进入老年代的原因**：经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象，而当复制对象时，大对象就意味着高额的内存复制开销。

    HotSpot 虚拟机提供了**-XX：PretenureSizeThreshold** 参数，指定大于该设置值的对象直接在老年代分配，这样做的目的就是避免在 Eden 区及两个 Survivor 区之间来回复制，产生大量的内存复制操作。

3. **长期存活的对象将进入老年代**

    对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，并且将其对象年龄设为1岁。对象在 Survivor 区中每熬过一次 Minor GC，年龄就增加1岁，当它的年龄增加到一定程度（默认为15，超过15时晋升），就会被晋升到老年代中。

    对象晋升老年代的年龄阈值， 可以通过参数 **-XX：MaxTenuringThreshold** 设置。

4. **动态对象年龄判断**

    虚拟机并不是永远要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代。

    如果在 Survivor 中**不超过某个年龄的所有对象大小的总和大于 Survivor 空间的一半**，则年龄大于或等于该年龄的对象可以直接进入老年代。

5. **空间分配担保**

    流程：

    1. 在发生 Minor GC 之前，虚拟机先**检查老年代最大可用的连续空间是否大于新生代所有对象总空间**，如果条件成立的话，那么 Minor GC 可以确认是安全的。
    2. 如果不成立， 则虚拟机会先查看 -XX： HandlePromotionFailure 参数的设置值是否允许担保失败（Handle Promotion Failure），不允许则进行一次 Full GC。
    3. 如果允许，那会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小。如果大于，将尝试进行一次Minor GC，尽管这次Minor GC是有风险的；如果小于则镜像一次 Full GC。
    4. 如果出现了担保失败，则重新发起一次 Full GC，这会停顿很长时间。

### 3.5 垃圾收集的分类

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E5%9E%83%E5%9C%BE%E6%94%B6%E9%9B%86%E5%88%86%E7%B1%BB.png" style="zoom: 80%;" />

#### 新生代收集(Minor GC/Young GC)

​	指目标只是新生代的垃圾收集，当 Eden 空间不足时将触发一次 Minor GC。

#### 老年代收集(Major GC/Old GC)

指目标只是老年代的垃圾收集。 目前**只有CMS收集器会有单独收集老年代的行为**。

> 另外请注意 =="Major GC" 这个说法现在有点混淆==， 在不同资料上常有不同所指，读者==需按上下文区分到底是指老年代的收集还是整堆收集==。==通常 Major GC 和 Full GC 是等价的，收集整个 GC 堆==。

#### 混合收集(Mixed GC)

指目标是**收集整个新生代以及部分老年代**的垃圾收集。 目前只有G1收集器会有这种行为。

#### 整堆收集(Full GC)

收集整个Java堆和方法区的垃圾收集。

**触发条件：**

1. **调用 System.gc()**

    只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。

2. **老年代空间不足**

    老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

    为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

3. **空间分配担保失败**

    使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

4. **JDK1.7 及以前的永久代空间不足**

    当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

    为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

5. **Concurrent Mode Failure**

    执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足（可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足），便会报 Concurrent Mode Failure 错误，并触发 Full GC。

### 3.6 经典垃圾收集器

|      收集器       | 串行、并行/并发 | 新生代/老年代 |      算法       |     目标     | 适用场景                                                     |
| :---------------: | :-------------: | :-----------: | :-------------: | :----------: | ------------------------------------------------------------ |
|      Serial       |      串行       |    新生代     |      复制       | 响应速度优先 | 单CPU环境、运行在客户端模式下的虚拟机                        |
|    Serial Old     |      串行       |    老年代     |   标记--整理    | 响应速度优先 | 供客户端模式下的HotSpot虚拟机使用、<br />作为CMS收集器发生失败时的后备预案 |
|      ParNew       |      并行       |    新生代     |      复制       | 响应速度优先 | 多CPU环境时运行在服务器模式下的虚拟<br />机，与CMS配合使用   |
| Parallel Scavenge |      并行       |    新生代     |      复制       |  吞吐量优先  | 适合在后台运算而不需要太多交互的分析任务                     |
|   Parallel Old    |      并行       |    老年代     |   标记--整理    | 响应速度优先 | 搭配 Parallel Scavenge ，用于注重吞吐<br />量或者处理器资源较为稀缺的场合 |
|        CMS        |      并发       |    老年代     |   标记--清除    | 响应速度优先 | 集中在互联网网站或者基于浏览器B/S系统的服务端上的 java 应用  |
|        G1         |      并发       |     both      | 标记--整理+复制 | 响应速度优先 | 面向服务端应用                                               |

注释：

- 串行：在执行垃圾收集的时候需要停顿用户程序，垃圾收集器与用户程序交替执行。
- 并行：多条垃圾收集线程并⾏⼯作，但此时⽤户线程仍然处于等待状态。  
- 并发：指⽤户线程与垃圾收集线程同时执⾏（但不⼀定是并⾏，可能会交替执⾏），⽤户程序在继续运⾏，⽽垃圾收集器运⾏在另⼀个CPU上。  

到jdk8为止，默认的垃圾收集器是Parallel Scavenge 和 Parallel Old，从jdk9开始，G1收集器成为默认的垃圾收集器。

#### CMS 收集器

CMS(Concurrent Mark Sweep)，基于标记 -- 清除算法实现，运行过程如下：

- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除：不需要停顿。

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

**缺点**：

- 吞吐量低
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。
- 使用标记 -- 清除算法会产生空间碎片

#### G1 收集器

​	G1 不再坚持固定大小以及固定数量的分代区域划分， 而是把连续的 Java 堆划分为多个大小相等的独立区域（Region） ， 每一个 Region 都可以根据需要， 扮演新生代的 Eden 空间、 Survivor 空间， 或者老年代空间。

​	收集器能够对扮演不同角色的 Region 采用不同的策略去处理， 这样无论是新创建的对象还是已经存活了一段时间、 熬过多次收集的旧对象都能获取很好的收集效果。  

​	G1 收集器在后台维护了⼀个优先列表，每次根据允许的收集时间，优先选择回收价值最⼤的 Region (这也就是它的名字Garbage-First 的由来)  

### 3.7 JVM常用参数

- **调整最大堆内存和最小堆内存**

    -Xmx –Xms：指定java堆最大值（默认值是物理内存的1/4(<1GB)）和初始java堆最小值（默认值是物理内存的1/64(<1GB))

    通常会将 -Xms 与 -Xmx两个参数的配置相同的值，**这样可以获得固定大小的堆内存，减少GC的次数和耗时，可以使得堆相对稳定**。

- **调整新生代和老年代的比例**

    -XX:NewRatio --- 老年代（不包含永久区）与新生代（eden+2*Survivor）的比例。

    例如：-XX:NewRatio=4，表示老年代 ： 新生代 = 4 ： 1，即新生代占整个堆的1/5。

- 调整 Survivor 区和 Eden 区的比例

    -XX:SurvivorRatio ---设置 Eden 区与两个 Survivor 区的比值。

    例如：-XX:SurvivorRatio=8，表示两个Survivor:eden=2:8，即一个Survivor占年轻代的1/10。

- **设置年轻代的大小**

    -XX:NewSize --- 设置年轻代大小

    -XX:MaxNewSize --- 设置年轻代最大值

<!-- 分页-->

<div style="page-break-after: always;"></div>

<!--分页-->

## 4. 类文件结构

```javascript
ClassFile {
	u4 magic; // 魔数，Class ⽂件的标志
    // class 的版本号
	u2 minor_version; // Class 的次版本号
	u2 major_version; // Class 的主版本号
    // 常量池，主要存放 字面量 和 符号引用
	u2 constant_pool_count; // 常量池容量
	cp_info constant_pool[constant_pool_count-1]; // 常量池数据，索引范围[1, constant_pool_count-1]
	
    u2 access_flags; // Class 的访问标记
	u2 this_class;  // 类索引
	u2 super_class;  // ⽗类索引
    // 接口索引集合
	u2 interfaces_count;  // 接⼝计数器
	u2 interfaces[interfaces_count];  // 接口数据
    // 字段集合
	u2 fields_count;  // 字段计数器
	field_info fields[fields_count];
    // 方法集合
	u2 methods_count;  // 方法计数器
	method_info methods[methods_count];
	// 属性集合
    u2 attributes_count;  // 属性计数器
	attribute_info attributes[attributes_count];
}
```

<!-- 分页-->

<div style="page-break-after: always;"></div>

<!--分页-->

## 5. 类加载机制

​	**类是在运行期间第一次使用时动态加载的**，而不是一次性加载所有类。因为如果一次性加载，那么会占用很多的内存。

### 5.1 类的生命周期

​	一个类型从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期将会经历**加载**(Loading)、**验证**(Verification)、**准备**(Preparation)、**解析**(Resolution)、**初始化**(Initialization)、**使用**(Using)和**卸载**(Unloading)七个阶段，其中**验证、准备、解析三个部分统称为连接**(Linking)。

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E7%B1%BB%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png" style="zoom:100%;" />

​	加载、 验证、 准备、 初始化和卸载这五个阶段的顺序是确定的，但**解析阶段可以在初始化阶段之后再开始**，这是为了支持 Java 语言的动态绑定特性。

### 5.2 类加载的过程

​	Java虚拟机中类加载的全过程为加载、 验证、 准备、 解析和初始化这五个阶段。  

#### 加载

​	“加载”阶段是整个“类加载”过程中的一个阶段，注意不要混淆。

​	在加载过程中，虚拟机需要完成以下三件事：

- 通过一个类的全限定名来获取定义此类的二进制字节流。 
- 将这个字节流所代表的静态存储结构转化为方法区的运行时数据结构。  
- 在内存中生成一个代表该类的 Class 对象，作为方法区中该类各种数据的访问入口。

​    **加载阶段与连接阶段的部分动作（ 如一部分字节码文件格式验证动作） 是交叉进行的**， 加载阶段尚未完成， 连接阶段可能已经开始。  

#### 验证

​	这一阶段的目的是确保 Class 文件的字节流中包含的信息符合《Java虚拟机规范》的全部约束要求，保证这些信息被当作代码运行后不会危害虚拟机自身的安全。  

#### 准备

​	准备阶段是正式**为类变量(即静态变量，被static修饰的变量)分配内存并设置类变量初始值**的阶段。

​	从概念上讲， 这些变量所使用的内存都应当在方法区中进行分配。**JDK7 及其之前，使用永久代实现方法区，是符合这种逻辑概念的；JDK8及其之后类变量则会随着Class对象一起存放在Java堆中**。

​	**注意：**

- 这时候进行内存分配的仅包括类变量， 而不包括实例变量， **实例变量将会在对象实例化时随着对象一起分配在Java堆中**。  

- 这里所说的初始值“通常情况”下是数据类型的零值，如果类变量是常量(被 static final修饰)，那么它将初始化为表达式所定义的值。

    ```java
    public static int value1 = 123; // 准备阶段将 value1 赋值为0
    public static final int value2 = 123;  // 准备阶段将 value2 赋值为123
    ```

#### 解析

​	解析阶段是Java虚拟机将常量池内的符号引用替换为直接引用的过程。

#### 初始化

​	直到初始化阶段， Java 虚拟机才真正开始执行类中编写的 Java 程序代码。进行准备阶段时，变量已经赋过一次系统要求的初始零值，而在初始化阶段，则会根据程序员通过程序编码制定的主观计划去初始化类变量和其他资源。**初始化阶段就是执行类构造器 \<clinit\>() 方法的过程**。

​	**注意：**

- 静态语句块中只能访问到定义在静态语句块之前的变量，**定义在它之后的变量， 在前面的静态语句块可以赋值，但是不能访问**。  
- `<clinit>()`方法不需要显式地调用父类构造器， Java虚拟机会保证在子类的`<clinit>()`方法执行前， 父类的`<clinit>()`方法已经执行完毕。 因此在Java虚拟机中第一个被执行的`<clinit>()`方法的类型肯定是`java.lang.Object`。由于父类的`<clinit>()`方法先执行， 也就意味着父类中定义的静态语句块要优先于子类的变量赋值操作。
- **如果一个类中没有静态语句块，也没有对变量的赋值操作，那么编译器可以不为这个类生成 `<clinit>() `方法**。  
- 接口与类一样也会生成 `<clinit>()` 方法，但与类不同的是执行接口的 `<clinit>()`方法不需要先执行父接口的 `<clinit>()`方法，**只有当父接口中定义的变量被使用时，父接口才会初始化**。实现接口的类在初始化的时候同样不会执行接口的 `<clinit>()` 方法。
- Java 虚拟机必须保证一个类的，`<clinit>()`方法在多线程环境中被正确地加锁同步， 如果多个线程同时去初始化一个类， 那么只会有其中一个线程去执行这个类的`<clinit>()`方法， 其他线程都需要阻塞等待， 直到活动线程执行完毕 `<clinit>()`方法。  

### 5.3 类加载的时机

​	《Java虚拟机规范》 中并没有进行强制约束什么时候进行“加载”，但是它严格规定了**==有且只有==六种情况必须立即对类进行“初始化”**（而加载、 验证、 准备自然需要在此之前开始）：  

1. 遇到 new、getstatic、putstatic 或 invokestatic 这四条字节码指令时，如果类型没有进行过初始化，则需要先触发其初始化阶段。能够生成这四条指令的典型Java代码场景有：
    - 使用new关键字实例化对象的时候
    - 读取或设置一个类型的静态字段
    - 调用一个类型的静态方法
2. 使用 java.lang.reflect 包的方法对类型进行反射调用的时候，如果类型没有进行过初始化，则需要先触发其初始化。
3. 当初始化类的时候，如果发现其父类还没有进行过初始化，则需要先触发其父类的初始化。
4. 当虚拟机启动时，用户需要指定一个要执行的主类（ 包含main()方法的那个类），虚拟机会先初始化这个主类。
5. 当使用 JDK 7新加入的动态语言支持时，如果一个 java.lang.invoke.MethodHandle 实例最后的解析结果为REF_getStatic、REF_putStatic、REF_invokeStatic、REF_newInvokeSpecial四种类型的方法句柄，并且这个方法句柄对应的类没有进行过初始化，则需要先触发其初始化。
6. 当一个接口中定义了JDK 8新加入的默认方法（ 被default关键字修饰的接口方法）时，如果有这个接口的实现类发生了初始化，那该接口要在其之前被初始化。

以上六种场景中的行为称为对一个类型进行**主动引用**。**除此之外，所有引用类型的方式都不会触发初始化，称为被动引用**。常见例子如下：

- 通过子类引用父类的静态字段，不会引起子类的初始化，只会触发父类的初始化。

    ```java
    public class SuperClass {
        static {
        	System.out.println("SuperClass init!");
        }
        public static int value = 123;
    }
    public class SubClass extends SuperClass {
        static {
            System.out.println("SubClass init!");
        }
    } 
    public class NotInitialization {
        public static void main(String[] args) {
            // 只有定义这个静态字段的类才会初始化
            System.out.println(SubClass.value);
        }
    }
    ```

- 通过数组定义来引用类，不会触发此类的初始化。

    ```java
    public class NotInitialization {
        public static void main(String[] args) {
            SuperClass[] sca = new SuperClass[10];
        }
    }
    ```

- 常量在编译阶段会存入调用类的常量池中，本质上并没有直接引用到定义常量的类，因此不会触发定义常量的类的初始化。

    ```java
    public class ConstClass {
        static {
            System.out.println("ConstClass init!");
        } 
        public static final String HELLOWORLD = "hello world";
    }
    
    public class NotInitialization {
        public static void main(String[] args) {
            System.out.println(ConstClass.HELLOWORLD);
        }
    }
    ```

### 5.4 类加载器与双亲委派模型

#### 类加载器

​	**比较两个类是否“相等”，只有在这两个类是由同一个类加载器加载的前提下才有意义**，否则，即使这两个类来源于同一个 Class 文件，被同一个 Java 虚拟机加载，只要加载它们的类加载器不同，那这两个类就必定不相等。  

​	JVM 中内置了三个重要的 ClassLoader，除了 BootstrapClassLoader 其他类加载器均由 Java 实现且全部继承⾃ java.lang.ClassLoader ：

-  **BootstrapClassLoader(启动类加载器)**，最顶层的加载类，由C++实现，负责加载 %JRE_HOME%/lib ⽬录下的 jar 包和类或者或被 -Xbootclasspath 参数指定的路径中的所有类。
-  **ExtensionClassLoader(扩展类加载器)** ，主要负责加载⽬录 %JRE_HOME%/lib/ext ⽬录下的 jar 包和类，或被 java.ext.dirs 系统变量所指定的路径下的jar包。  
-  **AppClassLoader(应⽤程序类加载器)** ，⾯向⽤户的加载器，负责加载当前应⽤ classpath 下的所有 jar 包和类。  

#### 双亲委派模型

​	双亲委派模型，即在类加载的时候，系统会⾸先判断当前类是否被加载过。已经被加载的类会直接返回，否则才会尝试加载。 加载的时候，**⾸先会把该请求委派该⽗类加载器去处理，因此所有的请求最终都应该传送到顶层的启动类加载器中。当⽗类加载器⽆法处理(它的搜索范围中没有找到所需的类 )时，子类加载器才会尝试⾃⼰来加载**。  

<img src="https://gitee.com/coldsun233/NotePic/raw/master/img/%E5%8F%8C%E4%BA%B2%E5%A7%94%E6%B4%BE%E6%A8%A1%E5%9E%8B.png" style="zoom:75%;" />

​	双亲委派模型要求除了顶层的启动类加载器外，其余的类加载器都应有自己的父类加载器。这里类加载器之间的父子关系一般不是以继承（Inheritance）的关系来实现的，而是通常使用组合（Composition）关系来复用父加载器的代码。  

**使用双亲委派模型的好处：**使得Java中的类随着它的类加载器一起具备了一种带有优先级的层次关系，这种层级关**可以避免类的重复加载，也保证了 Java 的核心 API 不会被篡改**。如果没有使用双亲委派模型，都由各个类加载器自行去加载的话，如果用户自己也编写了一个名为 java.lang.Object 的类，并放在程序的 ClassPath 中，那系统中就会出现多个不同的 Object 类，Java 类型体系中最基础的行为也就无从保证。  

双亲委派实现源码如下：

```java
public abstract class ClassLoader {
    // The parent class loader for delegation
    private final ClassLoader parent;
	...
    public Class<?> loadClass(String name) throws ClassNotFoundException {
        return loadClass(name, false);
    }

    protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);
            if (c == null) {
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);
                    } else {
                        c = findBootstrapClassOrNull(name);
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    c = findClass(name);
                }
            }
            if (resolve) {
                resolveClass(c);
            }
            return c;
        }
    }
	...
    // 自定义类加载器的话需要重新该函数
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        throw new ClassNotFoundException(name);
    }
}
```

### 5.5 补充

#### Java类的初始化顺序

正常类的初始化顺序：**静态变量/静态代码块 -> main方法 -> 非静态变量/代码块 -> 构造方法**

继承情况下的初始化顺序：
**父类–静态变量/父类–静态初始化块**
**子类–静态变量/子类–静态初始化块**
**父类–变量/父类–初始化块**
**父类–构造器**
**子类–变量/子类–初始化块**
**子类–构造器**

静态变量和静态初始化块的顺序取决于它们在类中出现的先后顺序，非静态变量和代码块的顺序同理。

#### main方法执行步骤

```java
public class Student {
    private String name;
    public Student(String name) {
        this.name = name;
    }
    public void sayName() {
        System.out.println("My name is " + name);
    }
}

public class Test {
    public static void main(String[] args) {
        Student student = new Student("jojo");
        student.sayName();
    }
}
```

以上述代码为例，执行过程为：

1. 编译好 Test.java 后得到 Test.class 后，执行 Test.class，系统会启动一个 JVM 进程，从 classpath 路径中找到一个名为 Test.class 的二进制文件，将 Test 的类信息加载到运行时数据区的方法区内，这个过程叫做 Test 类的加载

2. JVM 找到 Test的主程序入口，执行main方法

3. 这个main中的第一条语句为 Student student = new Student("jojo") ，就是让 JVM 创建一个Student对象，但是这个时候方法区中是没有 Student 类的信息的，所以 JVM 马上加载 Student 类，把 Student 类的信息放到方法区中
4. 加载完 Student 类后，JVM 在堆中为一个新的 Student 实例分配内存，然后调用构造函数初始化 Student 实例，这个 Student 实例持有指向方法区中的 Student 类的类型信息的引用
5. 执行student.sayName();时，JVM 根据 student 的引用找到 student 对象，然后根据 student 对象持有的引用定位到方法区中 student 类的类型信息的方法表，获得 sayName() 的字节码地址，然后执行 sayName()

<!-- 分页-->

<div style="page-break-after: always;"></div>

<!--分页-->