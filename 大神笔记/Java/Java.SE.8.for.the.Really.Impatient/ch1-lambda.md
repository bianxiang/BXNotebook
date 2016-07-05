[toc]

## 1. Lambda表达式

函数式编程适合处理并发和事件驱动编程。

- A lambda expression is a block of code with parameters.
- Lambda表达式可以被转换为**函数接口**
- Lambda表达式可以访问外围作用域的final变量
- 方法和构造器引用
- 可以向接口添加默认和静态方法，可以有具体实现

### 1.1 为什么使用Lambda？

一些传统的写法（可以被替换成Lambda表达式）：

    class Worker implements Runnable {
        public void run() {
            for (int i = 0; i < 1000; i++)
                doWork();
        }
        ...
    }

    Worker w = new Worker();
    new Thread(w).start();

----

    class LengthComparator implements Comparator<String> {
        public int compare(String first, String second) {
            return Integer.compare(first.length(), second.length());
        }
    }

    Arrays.sort(strings, new LengthComparator());

----

    button.setOnAction(new EventHandler<ActionEvent>() {
        public void handle(ActionEvent event) {
            System.out.println("Thanks for clicking!");
        }
    });


### 1.2 Lambda语法

一段比较两个字符串长度的Lambda表达式：

    (String first, String second) -> Integer.compare(first.length(), second.length())

Lambda表达式的形式是：参数，`->`箭头，表达式。如果语句有多个，可以像方法那样，用大括号包围，并显式使用`return`。例如：

    (String first, String second) -> {
        if (first.length() < second.length()) return -1;
        else if (first.length() > second.length()) return 1;
        else return 0;
    }

若Lambda表达式没有参数，仍需要写空的圆括号（就像方法那样）

    () -> { for (int i = 0; i < 1000; i++) doWork(); }

如果参数类型可以被推断，可以省略参数的类型。例如：

    Comparator<String> comp = (first, second) // Same as (String first, String second)
        -> Integer.compare(first.length(), second.length());

编译器根据`Comparator<String>`推断出`first`和`second`的类型。

如果只有一个参数且类型省略，则可以省略括号：

    EventHandler<ActionEvent> listener =
        event -> System.out.println("Thanks for clicking!");
    // Instead of (event) -> or (ActionEvent event) ->

可以向Lambda参数添加注解和`final`修饰符：

    (final String name) -> ...
    (@NonNull String name) -> ...

无法指定Lambda表达式的返回类型。返回类型总是从上下文中推断。

### 1.3 函数式接口

Java中已经有很多接口，目的仅是封装一块代码，如`Runnable`或`Comparator`。Lambda与这些接口兼容——若一个接口只有一个 **抽象** 方法，则使用这个接口的实例的地方可以用Lambda表达式替代。这样的接口称为**函数式接口**。

> 函数式接口的方法为什么必须是抽象的？难道接口中的方法不都是抽象的？接口可能重新声明`Object`类中的方法，如`toString`或`clone`，这些声明不是抽象的。（接口重新声明`Object`方法的目的之一是添加Javadoc，参见`Comparator`。）更重要的是，Java 8 允许接口声明非抽象方法（默认方法）。


Lambda表达式转换为接口的例子。`Arrays.sort`方法本期望一个`Comparator`对象。由于`Comparator`接口符合函数式接口条件。于是可以：

    Arrays.sort(words,
        (first, second) -> Integer.compare(first.length(), second.length()));

在表象之下，`Arrays.sort`方法仍收到一个实现`Comparator<String>`接口的类的对象。The management of these objects and classes is completely implementation dependent, 这种方式比传统的内部类高效的多。最好把Lambda表达式看做函数，而不是对象。

另一个例子：

    button.setOnAction(event -> System.out.println("Thanks for clicking!"));

实际上，**转换为函数式接口是Java中Lambda表达式的唯一用法**。有的语言支持声明函数类型，如`(String, String) -> int`，然后声明此类型的变量。但Java设计者坚持使用接口的概念。

> 不能将Lambda表达式赋给`Object`类型的变量——`Object`不是函数式接口。

`java.util.function`包下设计了很多通用的函数式接口（后面会详述）。例如，`BiFunction<T, U, R>`，声明函数取参数类型`T`和`U`，返回类型`R`。字符串比较的Lambda表达式就可以保存在此类型的变量中：

    BiFunction<String, String, Integer> comp
        = (first, second) -> Integer.compare(first.length(), second.length());

但这个变量不能用于排序——`Arrays.sort`不接受一个`BiFunction`。这样设计的目的是，接口表示的是一种特殊的目的，而不仅仅是提供什么方法。若想使用Lambda表达式，想清楚表达式的目的，并为它设计一个特殊的函数式接口。

`java.util.function`中的接口被几个Java 8 API使用。But keep in mind that you can equally well convert a lambda expression into a functional interface that is a part of whatever API you use today.

> 可以在任何函数式接口上标记`@FunctionalInterface`注解。好处是，编译器会检查接口是否只有一个抽象方法。Javadoc也会增加额外说明。此注解不强制使用。接口只要有单个抽象方法自动称为函数式接口，即使没有此注解。

如果Lambda表达式会抛出已检查异常，则相应的函数式接口的抽象方法中也需要声明异常。

    Runnable sleeper = () -> { System.out.println("Zzz"); Thread.sleep(1000); };
    // Error: Thread.sleep can throw a checked InterruptedException

### 1.4. 方法引用

有时可以用存在的方法完成某个功能。例如，你想打印事件：

    button.setOnAction(event -> System.out.println(event));

但更直接的方式直接将`println`方法传给`setOnAction`方法

    button.setOnAction(System.out::println);

`System.out::println`是一个**方法引用**（reference），与Lambda表达式`x -> System.out.println(x)`等价。

另一个例子。想要忽略大小写排序：

    Arrays.sort(strings, String::compareToIgnoreCase)

`::`分隔方法名和对象名/类名。三种情况：

- object::instanceMethod
- Class::staticMethod
- Class::instanceMethod

前两种情况等价于向方法提供参数的Lambda表达式。如`System.out::println`等价于`x ->  System.out.println(x)`。`Math::pow`等价于`(x, y) -> Math.pow(x, y)`。

第三种情况，第一个参数变成方法的目标。如`String::compareToIgnoreCase`等价于`(x, y) -> x.compareToIgnoreCase(y)`。

> 如果有重载方法，编译器将根据上下文判断。例如有两个版本的`Math.max`，分别用于int和double。

> 与Lambda表达式一样，方法引用也会被转换为函数式接口的实例。

可以在方法引用中捕获`this`参数。例如，`this::equals`与`x -> this.equals(x)`等价。也可以使用`super`。表达式`super::instanceMethod`使用`this`做目标调用父类版本方法。例子：

    class Greeter {
        public void greet() {
            System.out.println("Hello, world!");
        }
    }
    class ConcurrentGreeter extends Greeter {
        public void greet() {
            Thread t = new Thread(super::greet);
            t.start();
        }
    }


> In an inner class, you can capture the `this` reference of an enclosing class as `EnclosingClass.this::method` or `EnclosingClass.super::method`.

### 1.5.构造器引用

构造器引用类似于方法引用，只是方法名是`new`。如`Button::new`。假如有一组字符串，可以转换为一组按钮：

    List<String> labels = ...;
    Stream<Button> stream = labels.stream().map(Button::new);
    List<Button> buttons = stream.collect(Collectors.toList());

数组类型可以使用构造器引用。例如`int[]::new`是一个构造器引用，接收一个参数：数组长度；与`x -> new int[x]`等价。数组构造器引用用于解决Java的一个限制。无法构建泛型T的数组。表达式`new T[n]`是错误的，因为它会被擦除成`new Object[n]`。

例如，我们想返回一个按钮的数组。`Stream`接口有一个`toArray`方法：

    Object[] buttons = stream.toArray();

但它不满足要求。通过构造器引用`Button[]::new`就可以解决：

    Button[] buttons = stream.toArray(Button[]::new);

`toArray`方法调用这个构造器获得正确的数组。

### 1.6. 变量作用域

想在Lambda表达式中访问外层方法或类中的变量。如：

    public static void repeatMessage(String text, int count) {
        Runnable r = () -> {
            for (int i = 0; i < count; i++) {
                System.out.println(text);
                Thread.yield();
            }
        };
        new Thread(r).start();
    }

Lambda表达式的代码可能在`repeatMessage`返回后仍要运行，此时方法参数已消失。如何继续访问`text`和`count`？为理解，我们需要重新定义Lambda表达式。一个Lambda表达式由三部分构成：

1. 代码块
2. 参数
3. **自动变量**的值，即不是参数也不在Lambda代码中定义的变量。

在上面的例子中，`text`和`count`是自由变量。表示Lambda表达式的数据结构必须保存这些变量的**值**，即`"Hello"`和1000。

> 代码加自由变量的组合称为闭包。Lambda表达式是闭包。实际上。内部类从来都是闭包。

在lambda表达式中，只能引用值不改变的变量。例如下面的写法是无效的：

    public static void repeatMessage(String text, int count) {
        Runnable r = () -> {
            while (count > 0) {
                count--; // Error: Can't mutate captured variable
                System.out.println(text);
                Thread.yield();
            }
        };
        new Thread(r).start();
    }

这个限制的原因之一是，在Lambda表达式中修改变量不是线程安全的。

> 内部类也可以捕获外层作用域的值。Java 8之前，内部类只能访问final局部变量。此限制已被放松，以匹配Lambda表达式。内部类可以访问任何**有效final**的局部变量，即值不改变的变量。

The body of a lambda expression has the same scope as a nested block. The same rules for name conflicts and shadowing apply. It is illegal to declare a parameter or a local variable in the lambda that has the same name as a local variable.

    Path first = Paths.get("/usr/bin");
    Comparator<String> comp =
        (first, second) -> Integer.compare(first.length(), second.length());
    // Error: Variable first already defined

Inside a method, you can’t have two local variables with the same name, and therefore, you can’t introduce such variables in a lambda expression either.

在Launcher表达式中使用`this`关键字，`this`指向的创建Lambda表达式的方法中`this`的指向。例如：

    public class Application() {
        public void doWork() {
            Runnable runner = () -> {
                ...;
                System.out.println(this.toString());
                ...
            };
            ...
        }
    }

`this.toString()`中的this指`Application`对象，而不是`Runnable`对象。

### 1.7. 默认方法

很多编程语言将函数表达式与集合库集成。这样的代码比使用循环更简短易读。例如：

    for (int i = 0; i < list.size(); i++)
        System.out.println(list.get(i));

更好的方式：

    list.forEach(System.out::println);

但Java集合库很早就存在了。如果`Collection`接口增加新方法，如`forEach`，则所有定义了`Collection`实现类的程序都将出问题。

解决该问题的方法是，允许接口方法包含具体实现（称为默认方法）。这些方法可以被安全的添加到已存在的接口。

Java 8中，`forEach`已被添加到`Iterable`接口，它是`Collection`的父接口。

    interface Person {
        long getId();
        default String getName() { return "John Q. Public"; }
    }

实现类可以选择覆盖`getName`或保持原样。

若一个方法在一个接口中被声明为默认方法，而子类继承的其他接口或父类中有相同方法。这种情况，其他语言如Scala和C++有复杂的规则。但Java的规则很简单：

1. 父类胜出。如果父类提供了具体实现，相同名称且参数相同的**默认方法**被忽略。
2. 接口冲突。若两个接口有同名同参数类型方法，且其中一个或两个是**默认方法**, 则必须通过覆盖解决冲突。

现在看第二个规则。另一个接口也有`getName`方法：

    interface Named {
        default String getName() { return getClass().getName() + "_" + hashCode(); }
    }

若一个类实现这两个接口：

    class Student implements Person, Named {    ... }

类继承的两个`getName`并不一致。编译器将报错。需要在`Student`中覆盖方法解决冲突。在覆盖的方法中，可以选择任意版本，如：

    class Student implements Person, Named {
        public String getName() { return Person.super.getName(); }
        ...
    }

如果`Named`接口并未提供默认实现：

    interface Named {
        String getName();
    }

此时编译器仍视为有冲突，需要开发者解决。

> 当然若两个接口都未提供默认实现，不冲突。

另一种情况，类继承父类又实现接口。例如，

    class Student extends Person implements Named { ... }

此时，只有父类的方法起作用，接口中的默认方法被直接忽略。This is the "class wins" rule. The "class wins" rule ensures compatibility with Java 7.

> You can never make a default method that redefines one of the methods in the `Object` class. For example, you can’t define a default method for `toString` or `equals`, even though that might be attractive for interfaces such as `List`. As a consequence of the "classes win" rule, such a method could never win against `Object.toString` or `Object.equals`.

### 1.8. 接口中的静态方法

Java 8 允许向接口添加静态方法。It simply seemed to be against the spirit of interfaces as abstract specifications.

    public interface Path {
        public static Path get(String first, String... more) {
            return FileSystems.getDefault().getPath(first, more);
        }
        ...
    }

在Java 8，很多接口都添加了静态方法。例如`Comparator`接口增加了静态方法`comparing`。它接收一个函数产生一个比较器。函数从对象中取出用于比较的键。例如要按名称比较`Person`，使用`Comparator.comparing(Person::name)`。本章之前比较字符串长度的Launcher表达式`(first, second) -> Integer.compare(first.length(), second.length())`。可以简化为`Comparator.compare(String::length)`。The compare method turns a function (the key extractor) into a more complex function (the key-based comparator). We will examine such "higher order functions" in more detail in Chapter 3.

