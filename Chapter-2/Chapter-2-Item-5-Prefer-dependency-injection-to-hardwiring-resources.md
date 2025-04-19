## Chapter 2. Creating and Destroying Objects（创建和销毁对象）

### Item 5: Prefer dependency injection to hardwiring resources（依赖注入优于硬连接资源）

Many classes depend on one or more underlying resources. For example, a spell checker depends on a dictionary. It is not uncommon to see such classes implemented as static utility classes (Item 4):

许多类依赖于一个或多个底层资源。例如，拼写检查程序依赖于字典。常见做法是，将这种类实现为静态实用工具类（[Item-4](/Chapter-2/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)）：

```java
// Inappropriate use of static utility - inflexible & untestable!
public class SpellChecker {
    private static final Lexicon dictionary = ...;
    private SpellChecker() {} // Non-instantiable
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions(String typo) { ... }
}
```

Similarly, it’s not uncommon to see them implemented as singletons (Item 3):

类似地，我们也经常看到它们的单例实现（[Item-3](/Chapter-2/Chapter-2-Item-3-Enforce-the-singleton-property-with-a-private-constructor-or-an-enum-type.md)）:

```java
// Inappropriate use of singleton - inflexible & untestable!
public class SpellChecker {
    private final Lexicon dictionary = ...;
    private SpellChecker(...) {}
    public static INSTANCE = new SpellChecker(...);
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

Neither of these approaches is satisfactory, because they assume that there is only one dictionary worth using. In practice, each language has its own dictionary, and special dictionaries are used for special vocabularies. Also, it may be desirable to use a special dictionary for testing. It is wishful thinking to assume that a single dictionary will suffice for all time.

这两种方法都不令人满意，因为它们假设只使用一个字典。在实际应用中，每种语言都有自己的字典，特殊的字典用于特殊的词汇表。另外，最好使用一个特殊的字典进行测试。认为一本字典就足够了，是一厢情愿的想法。

You could try to have SpellChecker support multiple dictionaries by making the dictionary field nonfinal and adding a method to change the dictionary in an existing spell checker, but this would be awkward, error-prone,and unworkable in a concurrent setting. **Static utility classes and singletons are inappropriate for classes whose behavior is parameterized by an underlying resource.**

你可以尝试让 SpellChecker 支持多个字典：首先取消 dictionary 字段的 final 修饰，并在现有的拼写检查器中添加更改 dictionary 的方法。但是在并发环境中这种做法是笨拙的、容易出错的和不可行的。**静态实用工具类和单例不适用于由底层资源参数化的类。**

What is required is the ability to support multiple instances of the class (in our example, SpellChecker), each of which uses the resource desired by the client (in our example, the dictionary). A simple pattern that satisfies this requirement is to **pass the resource into the constructor when creating a new instance.** This is one form of dependency injection: the dictionary is a dependency of the spell checker and is injected into the spell checker when it is created.

所需要的是支持类的多个实例的能力（在我们的示例中是 SpellChecker），每个实例都使用客户端需要的资源（在我们的示例中是 dictionary）。满足此要求的一个简单模式是在**创建新实例时将资源传递给构造函数。** 这是依赖注入的一种形式：字典是拼写检查器的依赖项，在创建它时被注入到拼写检查器中。

```java
// Dependency injection provides flexibility and testability
public class SpellChecker {
    private final Lexicon dictionary;
    public SpellChecker(Lexicon dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }
    public boolean isValid(String word) { ... }
    public List<String> suggestions(String typo) { ... }
}
```

The dependency injection pattern is so simple that many programmers use it for years without knowing it has a name. While our spell checker example had only a single resource (the dictionary), dependency injection works with an arbitrary number of resources and arbitrary dependency graphs. It preserves immutability (Item 17), so multiple clients can share dependent objects(assuming the clients desire the same underlying resources). Dependency injection is equally applicable to constructors, static factories (Item 1), and builders (Item 2).

依赖注入模式非常简单，许多程序员在不知道其名称的情况下使用了多年。虽然拼写检查器示例只有一个资源（字典），但是依赖注入可以处理任意数量的资源和任意依赖路径。它保持了不可变性（[Item-17](/Chapter-4/Chapter-4-Item-17-Minimize-mutability.md)），因此多个客户端可以共享依赖对象（假设客户端需要相同的底层资源）。依赖注入同样适用于构造函数、静态工厂（[Item-1](/Chapter-2/Chapter-2-Item-1-Consider-static-factory-methods-instead-of-constructors.md)）和构建器（[Item-2](/Chapter-2/Chapter-2-Item-2-Consider-a-builder-when-faced-with-many-constructor-parameters.md)）。

A useful variant of the pattern is to pass a resource factory to the constructor.A factory is an object that can be called repeatedly to create instances of a type.Such factories embody the Factory Method pattern [Gamma95]. The `Supplier<T>` interface, introduced in Java 8, is perfect for representing factories. Methods that take a `Supplier<T>` on input should typically constrain the factory’s type parameter using a bounded wildcard type (Item 31) to allow the client to pass in a factory that creates any subtype of a specified type. For example, here is a method that makes a mosaic using a client-provided factory to produce each tile:

这种模式的一个有用变体是将资源工厂传递给构造函数。工厂是一个对象，可以反复调用它来创建类型的实例。这样的工厂体现了工厂方法模式 [Gamma95]。Java 8 中引入的 `Supplier<T>` 非常适合表示工厂。在输入中接受 `Supplier<T>` 的方法通常应该使用有界通配符类型（[Item-31](/Chapter-5/Chapter-5-Item-31-Use-bounded-wildcards-to-increase-API-flexibility.md)）来约束工厂的类型参数，以允许客户端传入创建指定类型的任何子类型的工厂。例如，这里有一个生产瓷砖方法，每块瓷砖都使用客户提供的工厂来制作马赛克：

```java
Mosaic create(Supplier<? extends Tile> tileFactory) { ... }
```

Although dependency injection greatly improves flexibility and testability, it can clutter up（使杂乱） large projects, which typically contain thousands of dependencies.This clutter can be all but eliminated by using a dependency injection framework, such as Dagger [Dagger], Guice [Guice], or Spring [Spring]. The use of these frameworks is beyond the scope of this book, but note that APIs designed for manual dependency injection are trivially adapted for（适用于） use by these frameworks.

尽管依赖注入极大地提高了灵活性和可测试性，但它可能会使大型项目变得混乱，这些项目通常包含数千个依赖项。通过使用依赖注入框架（如 Dagger、Guice 或 Spring），几乎可以消除这种混乱。这些框架的使用超出了本书的范围，但是请注意，为手动依赖注入而设计的 API 很容易被这些框架所使用。

In summary, do not use a singleton or static utility class to implement a class that depends on one or more underlying resources whose behavior affects that of the class, and do not have the class create these resources directly. Instead, pass the resources, or factories to create them, into the constructor (or static factory or builder). This practice, known as dependency injection, will greatly enhance the flexibility, reusability, and testability of a class.

总之，不要使用单例或静态实用工具类来实现依赖于一个或多个底层资源的类，这些资源的行为会影响类的行为，也不要让类直接创建这些资源。相反，将创建它们的资源或工厂传递给构造函数（或静态工厂或构建器）。这种操作称为依赖注入，它将大大增强类的灵活性、可复用性和可测试性。

---
**[Back to contents of the chapter（返回章节目录）](/Chapter-2/Chapter-2-Introduction.md)**
- **Previous Item（上一条目）：[Item 4: Enforce noninstantiability with a private constructor（用私有构造函数实施不可实例化）](/Chapter-2/Chapter-2-Item-4-Enforce-noninstantiability-with-a-private-constructor.md)**
- **Next Item（下一条目）：[Item 6: Avoid creating unnecessary objects（避免创建不必要的对象）](/Chapter-2/Chapter-2-Item-6-Avoid-creating-unnecessary-objects.md)**


### 扩展

在《Effective Java 第三版》中，**“依赖注入（Dependency Injection）优于硬连接资源（Hardwiring Resources）”** 这一节（条目5）通过一个具体的例子说明了如何通过依赖注入解耦代码、提高灵活性和可测试性。以下是该例子及其核心思想的总结：

---

#### **场景：实现一个拼写检查器（SpellChecker）**
假设需要开发一个拼写检查器，其功能依赖于一个词典（Dictionary）。以下是两种实现方式的对比：

##### **1. 硬连接资源（硬编码依赖）**
直接在 `SpellChecker` 类内部创建 `Dictionary` 对象，导致代码高度耦合：
```java
// 硬编码依赖的拼写检查器（不灵活）
public class SpellChecker {
    // 硬编码词典对象
    private static final Dictionary dictionary = new DefaultDictionary();

    // 私有构造方法，防止实例化
    private SpellChecker() {}

    public static boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public static List<String> suggestions(String typo) {
        return dictionary.getSimilarWords(typo);
    }
}
```
**问题**：
- `SpellChecker` 与 `DefaultDictionary` 强耦合，无法切换其他词典（如多语言词典）。
- 难以测试（例如，无法在测试中使用模拟词典）。

---

##### **2. 依赖注入（解耦依赖）**
将 `Dictionary` 作为参数传递给 `SpellChecker`，实现解耦：
```java
// 通过依赖注入解耦的拼写检查器
public class SpellChecker {
    private final Dictionary dictionary;

    // 注入词典依赖
    public SpellChecker(Dictionary dictionary) {
        this.dictionary = Objects.requireNonNull(dictionary);
    }

    public boolean isValid(String word) {
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        return dictionary.getSimilarWords(typo);
    }
}
```
**优势**：
- 灵活支持不同词典（例如：`EnglishDictionary`、`FrenchDictionary`）。
- 易于测试（可注入模拟词典 `MockDictionary`）。
- 符合单一职责原则（`SpellChecker` 只关注拼写逻辑，`Dictionary` 负责词典实现）。

---

#### **使用示例**
##### **客户端代码**
```java
// 创建不同的词典实现
Dictionary englishDict = new EnglishDictionary();
Dictionary frenchDict = new FrenchDictionary();

// 注入词典到拼写检查器
SpellChecker englishChecker = new SpellChecker(englishDict);
SpellChecker frenchChecker = new SpellChecker(frenchDict);

// 使用拼写检查器
englishChecker.isValid("color");    // 美式英语词典
frenchChecker.isValid("colour");     // 英式英语/法语词典
```

##### **单元测试**
```java
// 测试时注入模拟词典
public class SpellCheckerTest {
    @Test
    public void testIsValid() {
        // 创建模拟词典
        Dictionary mockDict = new Dictionary() {
            @Override
            public boolean contains(String word) {
                return "test".equals(word);
            }
        };

        SpellChecker checker = new SpellChecker(mockDict);
        assertTrue(checker.isValid("test"));
    }
}
```

---

#### **依赖注入的扩展形式**
书中进一步提到，依赖注入不仅限于构造函数注入，还可以通过以下方式实现：
1. **静态工厂方法**：
    ```java
    public static SpellChecker create(Dictionary dictionary) {
        return new SpellChecker(dictionary);
    }
    ```
2. **工厂对象**：
    ```java
    public interface DictionaryFactory {
        Dictionary getDictionary();
    }

    public SpellChecker(DictionaryFactory factory) {
        this.dictionary = factory.getDictionary();
    }
    ```
3. **依赖注入框架（如Spring、Guice）**：
    ```java
    // Spring示例：通过@Autowired注入
    @Service
    public class SpellChecker {
        @Autowired
        private Dictionary dictionary;
    }
    ```

---

#### **总结**
通过这个例子，**条目5**的核心思想是：
- **硬编码资源**会导致代码僵化、难以扩展和测试。
- **依赖注入**通过将依赖从类内部移到外部，实现了：
  - 解耦组件。
  - 提高代码灵活性和可重用性。
  - 简化单元测试。
- 依赖注入是控制反转（IoC）的一种实现方式，是编写可维护代码的重要原则。

该例子清晰展示了如何通过简单的构造函数参数传递实现解耦，是依赖注入的经典教学案例。