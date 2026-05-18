---
title: 8、java函数式接口
icon: material-icon-theme:bitbucket
createTime: 2026/05/18 15:25:52
permalink: /learning-notes/backend/java/java函数式接口/
order: 8
---


## 什么是函数式接口？

函数式接口是只包含一个抽象方法的接口。但是默认方法和静态方法在此接口中可以定义多个。Java 中的函数式接口可以被用作 Lambda 表达式的目标类型。通过函数式接口，可以实现更简洁、更具可读性的代码，从而支持函数式编程的思想。

Java 8 通过`@FunctionalInterface`注解来标识函数式接口。虽然这个注解不是强制性的，但建议使用它来确保接口符合函数式接口的定义。

```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void doSomething(); // 只有一个抽象方法
}
```

> 我们自己定义函数式接口的时候，@Functionallnterface是可以选的，就算我们不写这个注解，只要保证满足函数式接口定义的条件，也照样是函数式接口。但是，建议加上该注解;


## 常见的函数式接口

Java 中有一些内置的函数式接口，用于不同的用途：

- Runnable： 用于描述可以在单独线程中执行的任务。
- Callable： 类似于 Runnable，但可以返回执行结果或抛出异常。
- Comparator： 用于比较两个对象的顺序。
- Function： 接受一个参数并产生一个结果。
- Predicate： 接受一个参数并返回一个布尔值，用于判断条件是否满足。
- Supplier： 不接受参数，但返回一个值。


## 函数式接口作为参数

1、自定义函数式接口

```java
@FunctionalInterface
public interface CustomFunctionalInterface {
    void execute(String arg);
}
```


2、实现并且调用函数式接口

```java
public class TestFunctionalInterface {

    @Test
    public void testFunctionalInterface() {

        CustomFunctionalInterface customFunctionalInterface = str ->System.out.println("我是自定义接口函数，输出结果是:"+str);

        // 通过函数参数调用
        testFunctionalInterface2(customFunctionalInterface,"bugcode");
        //直接通过对象调用
        customFunctionalInterface.execute("bugcode");
    }
    // 定义一个以CustomFunctionalInterface 接口为参数的方法
    public void testFunctionalInterface2(CustomFunctionalInterface customFunctionalInterface,String str) {
        customFunctionalInterface.execute(str);
    }
}
```

在测试类 TestFunctionalInterface 中，定义了一个名为 testFunctionalInterface2 的方法，接受一个 CustomFunctionalInterface 参数，并在方法体中调用了 execute 方法。这个方法允许将任意实现了 CustomFunctionalInterface 的 Lambda 表达式传递进来，并执行其 execute 方法。


## 函数式接口作为方法的返回值

定义一个类（ComparatorDemo），在类中提供两个方法：
1. 一个方法是：Comparator< String > getComparator() 方法返回值Comparator是一个函数式接口
2. 一个方法是主方法，在主方法中调用getComparator方法

如果一个方法的返回值是一个函数式接口，我们可以把一个Lambda表达式作为结果返回


```java
public class ComparatorDemo {

    @Test
    public void testFunMethod(){
        ArrayList<String> list = new ArrayList<>();
        list.add("cccc");
        list.add("aa");
        list.add("b");
        list.add("ddd");

        System.out.println("排序前" + list);
        Collections.sort(list, getComparator());
        System.out.println("排序后" + list);
    }

    //如果一个方法的返回值是一个函数式接口，我们可以把一个Lambda表达式作为结果返回
    private static Comparator<String> getComparator() {
        //使用匿名内部类实现
//        return new Comparator<String>() {
//            @Override
//            public int compare(String s1, String s2) {
//                return s1.length()-s2.length();
//            }
//        };

        //Lambda表达式写法
        return (s1, s2) -> s1.length() - s2.length();
    }
}
```

Java 8在java.util.function包下预定义了大量的函数式接口供我们使用,我们重点学习下面4个接口

1. Supplier接口
2. Consumer接口
3. Predicate接口
4. Function接口

## Supplier

**函数签名:**

```java
@FunctionalInterface
public interface Supplier<T> {

    /**
     * Gets a result.
     *
     * @return a result
     */
    T get();
}
```

`Supplier< T >`：包含一个无参的方法

1. T get()：获得结果
2. 该方法不需要参数，他会按照某种实现逻辑（由Lambda表达式实现）返回一个数据
3. `Supplier< T >`接口也被称为生产型接口，如果我们指定了接口的泛型是什么类型，那么接口中的get方法就会产生什么类型的数据供我们使用

### 基于 Supplier 进行延迟计算

```java
public class SupplierExample {

    public static int generateRandomNumber(Supplier<Integer> supplier) {
        return supplier.get();
    }

    public static void main(String[] args) {
        // 实现匿名函数
        Supplier<Integer> randomSupplier = () -> new Random().nextInt(100);
        int randomNumber = generateRandomNumber(randomSupplier);
        System.out.println("Random number: " + randomNumber);
    }
}
```

在这个示例中，我们定义了一个 generateRandomNumber 方法，它接受一个 Supplier 参数，并通过调用 get 方法获取随机数。在 main 方法中，我们创建了一个随机数生成的 Supplier，然后将它传递给 generateRandomNumber 方法来获取随机数。


### 延迟初始化

```java
public class SupplierExample {

    private String expensiveResource = null;

    public String getExpensiveResource(Supplier<String> supplier) {
        if (expensiveResource == null) {
            expensiveResource = supplier.get();
        }
        return expensiveResource;
    }

    public static void main(String[] args) {
        Supplier<String> resourceSupplier = () -> {
            System.out.println("Initializing expensive resource...");
            return "Expensive Resource";
        };
        SupplierExample example = new SupplierExample();
        System.out.println(example.getExpensiveResource(resourceSupplier));
        System.out.println(example.getExpensiveResource(resourceSupplier));
    }

}

```

在这个示例中，我们定义了一个 getExpensiveResource 方法，它接受一个 Supplier 参数，并使用延迟初始化的方式获取昂贵的资源。在 main 方法中，我们创建了一个资源初始化的 Supplier，然后多次调用 getExpensiveResource 方法，观察只有在需要时才会初始化资源。


##  Consumer接口

函数签名:

```java
@FunctionalInterface
public interface Consumer<T> {

    /**
     * Performs this operation on the given argument.
     *
     * @param t the input argument
     */
    void accept(T t);

    /**
     * Returns a composed {@code Consumer} that performs, in sequence, this
     * operation followed by the {@code after} operation. If performing either
     * operation throws an exception, it is relayed to the caller of the
     * composed operation.  If performing this operation throws an exception,
     * the {@code after} operation will not be performed.
     *
     * @param after the operation to perform after this operation
     * @return a composed {@code Consumer} that performs in sequence this
     * operation followed by the {@code after} operation
     * @throws NullPointerException if {@code after} is null
     */
    default Consumer<T> andThen(Consumer<? super T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

`Consumer< T >`：包含两个方法

1. void accept(T t)：对给定的参数执行此操作
2. `default Consumer < T > andThen(Consumer after)`：返回一个组合的Consumer，依次执行此操作，然后执行after操作

`Consumer< T >`接口也被称为消费型接口，它消费的数据类型由泛型指定

### 基于 Consumer 进行筛选

```java
public class ConsumerExample {

    public static void processIntegers(List<Integer> list, Consumer<Integer> consumer) {
        for (Integer num : list) {
            consumer.accept(num);
        }
    }

    public static void main(String[] args) {
        List<Integer> integerList = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 0);
        Consumer<Integer> squareAndPrint = num -> {
            int square = num * num;
            System.out.println("Square of " + num + " is: " + square);
        };
        processIntegers(integerList, squareAndPrint);
    }
}
```


在这个示例中，我们定义了一个 processIntegers 方法，它接受一个整数列表和一个 Consumer 参数，并在方法内遍历列表，对每个元素执行 accept 方法。在 main 方法中，我们创建了一个 Consumer 实现 squareAndPrint，它会计算每个元素的平方并打印出来。

## Predicate接口

当涉及到对集合或数据进行筛选时，Java 中的函数式接口 Predicate 可以发挥重要作用。Predicate 是一个通用的函数式接口，用于定义一个接受参数并返回布尔值的操作，用于判断条件是否满足。

**函数签名**

```java
@FunctionalInterface
public interface Predicate<T> {

    /**
     * Evaluates this predicate on the given argument.
     *
     * @param t the input argument
     * @return {@code true} if the input argument matches the predicate,
     * otherwise {@code false}
     */
    boolean test(T t);

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * AND of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code false}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ANDed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * AND of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> and(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }

    /**
     * Returns a predicate that represents the logical negation of this
     * predicate.
     *
     * @return a predicate that represents the logical negation of this
     * predicate
     */
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }

    /**
     * Returns a composed predicate that represents a short-circuiting logical
     * OR of this predicate and another.  When evaluating the composed
     * predicate, if this predicate is {@code true}, then the {@code other}
     * predicate is not evaluated.
     *
     * <p>Any exceptions thrown during evaluation of either predicate are relayed
     * to the caller; if evaluation of this predicate throws an exception, the
     * {@code other} predicate will not be evaluated.
     *
     * @param other a predicate that will be logically-ORed with this
     *              predicate
     * @return a composed predicate that represents the short-circuiting logical
     * OR of this predicate and the {@code other} predicate
     * @throws NullPointerException if other is null
     */
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }

    /**
     * Returns a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}.
     *
     * @param <T> the type of arguments to the predicate
     * @param targetRef the object reference with which to compare for equality,
     *               which may be {@code null}
     * @return a predicate that tests if two arguments are equal according
     * to {@link Objects#equals(Object, Object)}
     */
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
}
```

`Predicate< T >`：常用的四个方法

- `boolean test(T t)`：对给定的参数进行判断（判断逻辑由Lambda表达式实现），返回一个布尔值
- `default Predicate< T > negate()`：返回一个逻辑的否定，对应逻辑非
- `default Predicate< T > and()`：返回一个组合判断，对应短路与
- `default Predicate< T > or()`：返回一个组合判断，对应短路或
- `isEqual()`：测试两个参数是否相等
- `Predicate< T >`：接口通常用于判断参数是否满足指定的条件


### 基于 Predicate 进行筛选

```java
public class PredicateExample {

    public static List<String> filterStrings(List<String> list, Predicate<String> predicate) {
        List<String> filteredList = new ArrayList<>();
        for (String str : list) {
            if (predicate.test(str)) {
                filteredList.add(str);
            }
        }
        return filteredList;
    }

    public static void main(String[] args) {
        List<String> stringList = Arrays.asList("apple", "banana", "cherry", "date", "elderberry");
        Predicate<String> lengthPredicate = str -> str.length() > 5;
        List<String> longStrings = filterStrings(stringList, lengthPredicate);
        System.out.println("Long strings: " + longStrings);
    }

}
```

在这个示例中，我们定义了一个 filterStrings 方法，它接受一个字符串列表和一个 Predicate 参数，并返回符合条件的字符串列表。在 main 方法中，我们创建了一个长度判断的 Predicate，然后使用它来筛选出长度大于 5 的字符串。

## Function接口

Function 接口定义了一个名为 apply 的抽象方法，接受一个参数并返回一个结果。这个接口用于表示一个对输入数据的转换操作。

函数签名:

```java
@FunctionalInterface
public interface Function<T, R> {

    /**
     * Applies this function to the given argument.
     *
     * @param t the function argument
     * @return the function result
     */
    R apply(T t);

    /**
     * Returns a composed function that first applies the {@code before}
     * function to its input, and then applies this function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of input to the {@code before} function, and to the
     *           composed function
     * @param before the function to apply before this function is applied
     * @return a composed function that first applies the {@code before}
     * function and then applies this function
     * @throws NullPointerException if before is null
     *
     * @see #andThen(Function)
     */
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    /**
     * Returns a composed function that first applies this function to
     * its input, and then applies the {@code after} function to the result.
     * If evaluation of either function throws an exception, it is relayed to
     * the caller of the composed function.
     *
     * @param <V> the type of output of the {@code after} function, and of the
     *           composed function
     * @param after the function to apply after this function is applied
     * @return a composed function that first applies this function and then
     * applies the {@code after} function
     * @throws NullPointerException if after is null
     *
     * @see #compose(Function)
     */
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    /**
     * Returns a function that always returns its input argument.
     *
     * @param <T> the type of the input and output objects to the function
     * @return a function that always returns its input argument
     */
    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

在上述定义中，T 表示输入类型，R 表示输出类型。

`Runction<T,R>`：常用的两个方法

`R apply(T t)`：将此函数应用于给定的参数
`default< V >：Function andThen(Function after)`：返回一个组合函数，首先将该函数应用于输入，然后将after函数应用于结果
`Function<T,R>`：接口通常用于对参数进行处理，转换（处理逻辑由Lambda表达式实现），然后返回一个新值


### 大小写转换

```java
public class FunctionExample {

    public static String convertToUpperCase(String input, Function<String, String> function) {
        return function.apply(input);
    }
    public static void main(String[] args) {
        String original = "hello world";
        String upperCase = convertToUpperCase(original, str -> str.toUpperCase());
        System.out.println(upperCase);
    }
}

```

在这个示例中，我们定义了一个 convertToUpperCase 方法，它接受一个字符串和一个 Function 参数，用于将输入字符串转换为大写。在 main 方法中，我们通过传递一个 Function 实现来执行转换操作。


### 字符串长度映射

```java
public class FunctionExample {

    public static List<Integer> mapStringLengths(List<String> list, Function<String, Integer> function) {
        return list.stream()
                .map(function)
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        List<String> strings = List.of("apple", "banana", "cherry", "date");
        List<Integer> lengths = mapStringLengths(strings, str -> str.length());
        System.out.println(lengths);
    }
}
```

在这个示例中，我们定义了一个 mapStringLengths 方法，它接受一个字符串列表和一个 Function 参数，用于将输入字符串映射为它们的长度。通过使用 map 操作，我们在列表中的每个字符串上执行了长度映射。