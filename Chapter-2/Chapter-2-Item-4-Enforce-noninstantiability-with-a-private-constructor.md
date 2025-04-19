## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 4: Enforce noninstantiability with a private constructor（用私有构造函数实施不可实例化）

Occasionally you’ll want to write a class that is just a grouping of static methods and static fields. Such classes have acquired a bad reputation because some people abuse them to avoid thinking in terms of objects, but they do have valid uses. They can be used to group related methods on primitive values or arrays, in the manner of java.lang.Math or java.util.Arrays. They can also be used to group static methods, including factories (Item 1), for objects that implement some interface, in the manner of java.util.Collections. (As of Java 8, you can also put such methods in the interface, assuming it’s yours to modify.) Lastly, such classes can be used to group methods on a final class, since you can’t put them in a subclass.

有时你会想要写一个类，它只是一个静态方法和静态字段的组合。这样的类已经获得了坏名声，因为有些人滥用它们来避免从对象角度思考，但是它们确有用途。它们可以用 java.lang.Math 或 java.util.Arrays 的方式，用于与原始值或数组相关的方法。它们还可以用于对以 java.util.Collections 的方式实现某些接口的对象分组静态方法，包括工厂（[Item-1](/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）。（对于 Java 8，你也可以将这些方法放入接口中，假设你可以进行修改。）最后，这些类可用于对 final 类上的方法进行分组，因为你不能将它们放在子类中。

Such utility classes were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler provides a public, parameterless default constructor. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

这样的实用程序类不是为实例化而设计的：实例是无意义的。然而，在没有显式构造函数的情况下，编译器提供了一个公共的、无参数的默认构造函数。对于用户来说，这个构造函数与其他构造函数没有区别。在已发布的 API 中看到无意中实例化的类是很常见的。

**Attempting to enforce noninstantiability by making a class abstract does not work.** The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19). There is, however, a simple idiom to ensure noninstantiability. A default constructor is generated only if a class contains no explicit constructors, so **a class can be made noninstantiable by including a private constructor:**

**译注：原文 noninstantiable 应修改为 non-instantiable ，译为「不可实例化的」**

**试图通过使类抽象来实施不可实例化是行不通的。** 可以对类进行子类化，并实例化子类。此外，它误导用户认为类是为继承而设计的（[Item-19](/Chapter-4/Chapter-4-Item-19-Design-and-document-for-inheritance-or-else-prohibit-it.md)）。然而，有一个简单的习惯用法来确保不可实例化。只有当类不包含显式构造函数时，才会生成默认构造函数，因此**可以通过包含私有构造函数使类不可实例化：**

```java
// Noninstantiable utility class
public class UtilityClass {
    // Suppress default constructor for noninstantiability
    private UtilityClass() {
        throw new AssertionError();
    } ... // Remainder omitted
}
```

Because the explicit constructor is private, it is inaccessible outside the class.The AssertionError isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances. This idiom is mildly counterintuitive because the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a comment, as shown earlier.

因为显式构造函数是私有的，所以在类之外是不可访问的。AssertionError 不是严格要求的，但是它提供了保障，以防构造函数意外地被调用。它保证类在任何情况下都不会被实例化。这个习惯用法有点违反常规，因为构造函数是明确提供的，但不能调用它。因此，如上述代码所示，包含注释是明智的做法。

As a side effect, this idiom also prevents the class from being subclassed. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.

这个习惯用法也防止了类被子类化，这是一个副作用。所有子类构造函数都必须调用超类构造函数，无论是显式的还是隐式的，但这种情况下子类都没有可访问的超类构造函数可调用。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 3: Enforce the singleton property with a private constructor or an enum type（使用私有构造函数或枚举类型实施单例属性）](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)**
- **Next Item（下一条目）：[Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）](/Chapter-2/Chapter-2-Item-5-Prefer-dependency-injection-to-hardwiring-resources.md)**



### 扩展-类不需要实例化的场景

在 Java 中，如果一个类**不需要被实例化**（即不允许创建该类的对象），可以通过将其设计为**不可实例化**的类。以下是需要这种能力的典型场景及实现方法：

---

#### **1. 工具类（Utility Class）**
**场景**：类仅包含静态方法或静态常量，无需实例化即可使用。  
**示例**：`Math`、`Collections`、`Arrays` 等工具类。  
**实现**：私有化构造方法，禁止通过 `new` 创建对象。  
```java
public final class StringUtils {
    // 私有构造方法，防止实例化
    private StringUtils() {
        throw new AssertionError("工具类不可实例化");
    }

    public static boolean isEmpty(String str) {
        return str == null || str.isEmpty();
    }
}
```

---

#### **2. 常量类**
**场景**：类仅用于定义常量，不需要实例。  
**示例**：系统配置参数、枚举常量集合。  
**实现**：私有构造方法 + `final` 修饰类。  
```java
public final class AppConstants {
    private AppConstants() {} // 防止实例化

    public static final int MAX_RETRY = 3;
    public static final String DEFAULT_ENCODING = "UTF-8";
}
```

---

#### **3. 抽象类（Abstract Class）**
**场景**：类需要被继承，但自身不提供完整实现。  
**示例**：框架中的基类（如 `AbstractList`）。  
**实现**：通过 `abstract` 关键字声明抽象类。  
```java
public abstract class AbstractParser {
    public abstract void parse(String input);
}
```

---

#### **4. 接口的默认方法伴随类**
**场景**：接口的默认方法需要共享公共逻辑，但实现类不依赖实例状态。  
**示例**：`java.util.stream.Collectors`（伴随 `Collector` 接口）。  
**实现**：私有构造方法 + 静态方法。  
```java
public interface Collector<T, A, R> {
    // 接口方法
}

public final class Collectors {
    private Collectors() {} // 不可实例化

    public static <T> Collector<T, ?, List<T>> toList() {
        return new CollectorImpl<>(...);
    }
}
```

---

#### **5. 单例模式（Singleton）**
**场景**：全局只允许存在一个实例。  
**示例**：数据库连接池、配置管理器。  
**实现**：私有构造方法 + 静态实例。  
```java
public class DatabasePool {
    private static final DatabasePool INSTANCE = new DatabasePool();

    private DatabasePool() {} // 私有构造方法

    public static DatabasePool getInstance() {
        return INSTANCE;
    }
}
```

---

#### **6. 防止反射/反序列化攻击**
**场景**：防止通过反射或反序列化绕过构造方法限制。  
**实现**：在私有构造方法中抛出异常。  
```java
public class SecurityUtils {
    private SecurityUtils() {
        // 防止通过反射实例化
        throw new UnsupportedOperationException("不可实例化");
    }
}
```

---

#### **关键实现方法**
1. **私有构造方法**：阻止通过 `new` 关键字实例化。  
2. **`final` 类**：防止子类化（可选）。  
3. **抛出异常**：防御反射攻击。  

---

#### **总结**
需要不可实例化的场景：  
- 工具类/常量类：无需状态，直接通过类名调用方法。  
- 抽象类/接口伴随类：设计上不需要实例。  
- 单例模式：严格限制实例数量。  
- 安全性需求：防止反射或反序列化攻击。  

通过合理使用不可实例化设计，可以提升代码的**安全性**、**可维护性**和**表达清晰度**。