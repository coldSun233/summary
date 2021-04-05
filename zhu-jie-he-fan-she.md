# 注解和反射

## 注解Annotation

注解可以理解为代码里的特殊标记，这些标记可以在编译，类加载，运行时被读取，并执行相应的处理。

### 内置注解

* @Override - 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
* @Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
* @SuppressWarnings - 指示编译器去忽略注解中声明的警告，需要传值，例如@suppressWarnings\("all"\)。
* @SafeVarargs - Java 7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
* @FunctionalInterface - Java 8 开始支持，标识一个匿名函数或函数式接口。

### 元注解

作用于其他注解的注解

* **@Retention**

  Retention英文意思有保留、保持的意思，它**表示注解存在阶段是保留在源码（编译期），字节码（类加载）或者运行期（JVM中运行）**。在@Retention注解中使用枚举RetentionPolicy来表示注解保留时期。

  @Retention\(RetentionPolicy.SOURCE\)，注解仅存在于源码中，在class字节码文件中不包含

  @Retention\(RetentionPolicy.CLASS\)， 默认的保留策略，注解会在class字节码文件中存在，但运行时无法获得

  @Retention\(RetentionPolicy.RUNTIME\)， 注解会在class字节码文件中存在，在运行时可以通过反射获取到

* **@Documented** - 标记这些注解是否包含在用户文档中。
* **@Target**

  Target的英文意思是目标，这也很容易理解，使用@Target元注解**表示的注解作用的范围**，可以是类，方法，方法参数变量等，同样也是通过枚举类ElementType表达作用类型。

  @Target\(ElementType.TYPE\) 作用接口、类、枚举、注解

  @Target\(ElementType.FIELD\) 作用属性字段、枚举的常量

  @Target\(ElementType.METHOD\) 作用方法

  @Target\(ElementType.PARAMETER\) 作用方法参数

  @Target\(ElementType.CONSTRUCTOR\) 作用构造函数

  @Target\(ElementType.LOCAL\_VARIABLE\)作用局部变量

  @Target\(ElementType.ANNOTATION\_TYPE\)作用于注解（@Retention注解中就使用该属性）

  @Target\(ElementType.PACKAGE\) 作用于包

  @Target\(ElementType.TYPE\_PARAMETER\) 作用于类型泛型，即泛型方法、泛型类、泛型接口 （jdk1.8加入）

  @Target\(ElementType.TYPE\_USE\) 类型使用.可以用于标注任意类型除了 class （jdk1.8加入）

* **@Inherited**

  被@Inherited注解了的注解修饰了一个父类，如果他的子类没有被其他注解修饰，则它的子类也继承了父类的注解。

  这个注解指定被他修饰的注解将具有继承性——如果某个类使用了@Xxx，则其子类将自动被@Xxx修饰

* **@Repeatable**

  Repeatable的英文意思是可重复的。顾名思义说明被这个元注解修饰的注解可以同时作用一个对象多次，但是每次作用注解又可以代表不同的含义。

### 自定义注解

使用`@interface`自定义注解时，自动继承`Annotation`接口

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface 注解名{
    // 注解的参数 参数类型 参数名+();
    int num(); // 这是一个参数，不是一个方法
    // 也可以给默认值
    String name() default "";
}
```

## 反射机制

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类中的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 获得Class类的几种方式

* 若已知具体的类，通过类的`class`属性获取，该方法最为安全可靠，程序性能最高

  ```java
    Class clazz = Person.class;
  ```

* 已知某个类的实例，调用该实例的`getClass()`方法获取Class对象

  ```java
    Class clazz = person.getClass();
  ```

* 已知一个类的全类名，且该类在类路径下，可通过Class类的静态方法f`orName()`获取，可能抛出`ClassNotFoundException`

  ```java
    Class clazz = Class.forName("come.example.Person");
  ```

* 内置基本数据内型的包装类可以直接用类名.Type
* 利用ClassLoader

### 拥有Class对象的类型

```java
Class c1 = Object.class;  // 类
Class c2 = Comparable.class;  // 接口
Class c3 = String[].class;  // 一维数组
Class c4 = int[][].class;  // 二维数组
Class c5 = Override.class;  // 注解
Class c6 = ElementType.class; // 枚举
Class c7 = Integer.class;  // 基本数据类型
Class c8 = void.class;  // void
Class c9 = Class.class; // Class
// 无论容量大小，只要元素类型与维度相同，数组的Class就是同一个Class
int[] a = new int[10];
int[] b = new int[100];
// a.getClass() == b.getClass()
```

### 获取类的运行时数据结构

Class类的实例方法

```java
public String getName() // 返回包名+类名
public String getSimpleName() // 返回类名
```

```java
public Field[] getFields() throws SecurityException
```

返回包含一个数组Field对象反射由此表示的类或接口的所有可访问的公共字段类对象。如果此类对象表示没有无法访问的公共字段的类或接口，则此方法返回长度为0的数组。

如果这个类对象表示一个类，那么**这个方法返回该类及其所有超类的公共字段**。

```java
public Field[] getDeclaredFields() throws SecurityException
```

返回的数组Field对象反映此表示的类或接口声明的所有字段类对象。这**包括公共，受保护，默认（包）访问和私有字段，但==不包括继承的字段==**。

```java
public Method[] getMethods() throws SecurityException
```

返回包含一个数组方法对象反射由此表示的类或接口的所有公共方法类对象，**包括那些由类或接口和那些从超类和超接口继承的声明**。

```java
public Method[] getDeclaredMethods() throws SecurityException
```

返回包含一个数组方法对象反射的类或接口的所有声明的方法，通过此表示类对象，包括公共，保护，默认（包）访问和私有方法，但**不包括继承的方法**。

### 动态创建对象执行方法

```java
Class student = Class.forName("come.example.Student");
// 通过Class创建对象，本质是调用无参构造器
Student s0 = (Student)student.newInstance();

// 通过构造器创建对象
Constructor constructor = student.getDeclaredConstructor(String.class, int.class, int.class);
Student s1 = (Student)constructor.newInstance("jojo", 1, 18);

// 通过反射调用普通方法
Student s2 = (Student)student.newInstance();
Method setName = student.getDeclaredMethod("setName", String.class);  // setName没有参数设置为null
// setName.setAccessible(true); 如果方法是私有的需要这一行
setName.invoke(s2, "tom"); // invoke(对象, 方法的参数)，无参数设置为null

// 通过反射操作属性
Student s3 = (Student)student.newInstance();
Field name = student.getDeclaredFiedl("name");
name.setAccessible(true); // 想要访问私有属性需要设置true
name.set("tom");
name.get();
```

### 反射操作泛型

Java中的泛型仅仅只是给编译器Javac使用的，确保数据的安全性和免去强制类型转换的问题，但是，一旦编译完成，所有和泛型有关的类型全部擦除。

为了通过反射操作泛型，Java新增了几种类型

* ParameterizedType    表示一种参数化的类型 , 比如 Collection\,可以获取 String信息
* GenericArrayType    泛型数组类型
* TypeVariable    各种类型变量的公共父接口
* WildcardType    代表一种通配符类型表达式，比如? extends Number,? super Integer \(Wildcard 是一个单词，就是通配符\)

```java
public void test01(Map<String, Person> map, List<Person> list){
        System.out.println("Text.test01()");
    }
public Map<Integer, Person> test02(){
    System.out.println("Text.test02()");
    return null;
}
//泛型参数
Method method = clazz.getMethod("test01", Map.class, List.class);
// 得到Map<String, Person>和List<Person>
Type[] genericParameterTypes = method.getGenericParameterTypes();
for(Type genericParameterType : genericParameterTypes) {
    if (genericParameterType instanceof ParameterizedType) {
       // 得到Map<String, Person>中的String和Person
        Type[] actualTypeArguments =
            ((ParameterizedType)genericParameterType).getActualTypeArguments();
        for (Type actualTypeArgument : actualTypeArguments) {
            System.out.println(actualTypeArgument);
        } 
    }
}
// 泛型返回值
Method method = clazz.getMethod("test02", null);
// 得到Map<Integer, Person>
Type genericReturnType = method.getGenericReturnType();
if (genericReturnType instanceof ParameterizedType) {
    // 得到Map<Integer, Person>中的Integer和Person
    Type[] actualTypeArguments =
        ((ParameterizedType)genericReturnType).getActualTypeArguments();
    for (Type actualTypeArgument : actualTypeArguments) {
        System.out.println(actualTypeArgument);
    } 
}
```

### 反射操作注解

Class类的一些实例方法

```text
public Annotation[] getAnnotations()
```

返回此元素上存在的注解。

```text
public <A extends Annotation> A getAnnotation(Class<A> annotationClass)
```

返回指定类型的注解，不存在则返回null

```java
// 一个类注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@intarface ClassAnnotation {
    String value();
}
// 一个属性的注解
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@intarface FieldAnnotation {
    String columnName();
    String type();
    int length();
}
// 获得注解的value
ClassAnnotation cAnnotation = (ClassAnnotation)clazz.getAnnotation(ClassAnnotation.class);
cAnnotation.value();// 返回value的值

Field fieldName = clazz.getField("fieldName");
FieldAnnotation fAnnotation = (FieldAnnotation)fieldName.getAnnotation(FieldAnnotation.class);
fAnnotation.columnName();//返回columnName的值
fAnnotation.type();//返回type的值
fAnnotation.length();//返回length的值
```

