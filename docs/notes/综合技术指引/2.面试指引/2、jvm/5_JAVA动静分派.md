---
title: 5、Java动态分派和静态分派
icon: catppuccin:java-exception
order: 5
author: bugcode
date: 2024-11-16T00:00:00.000Z
copyright: bugcode
createTime: 2025/09/04 11:14:44
permalink: /compre-guide/interview/jvm/Java动态分派和静态分派/
---

# Java 中的分派（Dispatch）机制详解

## 一、分派的基本概念

### 1.1 什么是分派？

分派（Dispatch）是指确定调用哪个方法的过程。Java 中的分派机制决定了在运行时如何选择正确的方法实现。

```java
public class DispatchBasic {
    public static void main(String[] args) {
        // 分派要解决的问题：
        // 当调用 obj.method() 时，应该执行哪个方法？
        Animal animal = new Dog();  // 编译时类型是Animal，运行时类型是Dog
        animal.speak();  // 这里需要分派机制决定调用Animal.speak()还是Dog.speak()
    }
}

class Animal {
    void speak() {
        System.out.println("Animal speaks");
    }
}

class Dog extends Animal {
    @Override
    void speak() {
        System.out.println("Dog barks");
    }
}
```

### 1.2 分派的分类

Java 中的分派主要分为两种：

1. **静态分派（Static Dispatch）**：在编译期确定方法版本
2. **动态分派（Dynamic Dispatch）**：在运行期确定方法版本

## 二、静态分派（Static Dispatch）

### 2.1 什么是静态分派？

静态分派发生在编译阶段，根据变量的**声明类型**（静态类型）来确定调用的方法。

```java
public class StaticDispatch {
    
    static abstract class Human {
    }
    
    static class Man extends Human {
    }
    
    static class Woman extends Human {
    }
    
    // 重载方法
    public void sayHello(Human guy) {
        System.out.println("hello,guy!");
    }
    
    public void sayHello(Man guy) {
        System.out.println("hello,gentleman!");
    }
    
    public void sayHello(Woman guy) {
        System.out.println("hello,lady!");
    }
    
    public static void main(String[] args) {
        // 静态类型和实际类型
        Human man = new Man();      // 静态类型：Human，实际类型：Man
        Human woman = new Woman();  // 静态类型：Human，实际类型：Woman
        
        StaticDispatch sr = new StaticDispatch();
        
        // 静态分派：根据静态类型Human选择方法
        sr.sayHello(man);    // 输出: hello,guy!
        sr.sayHello(woman);  // 输出: hello,guy!
        
        // 明确指定静态类型
        sr.sayHello((Man) man);     // 输出: hello,gentleman!
        sr.sayHello((Woman) woman); // 输出: hello,lady!
    }
}
```

### 2.2 方法重载与静态分派

方法重载（Overload）是静态分派的典型应用：

```java
public class OverloadExample {
    
    // 重载方法
    public void test(String str) {
        System.out.println("String: " + str);
    }
    
    public void test(Object obj) {
        System.out.println("Object: " + obj);
    }
    
    public void test(Integer num) {
        System.out.println("Integer: " + num);
    }
    
    public void test(int num) {
        System.out.println("int: " + num);
    }
    
    public void test(char c) {
        System.out.println("char: " + c);
    }
    
    public void test(Character c) {
        System.out.println("Character: " + c);
    }
    
    public static void main(String[] args) {
        OverloadExample example = new OverloadExample();
        
        // 静态分派规则：
        // 1. 精确匹配优先
        example.test("hello");      // String: hello
        
        // 2. 自动类型转换（向上转型）
        Object obj = "world";
        example.test(obj);          // Object: world
        
        // 3. 自动装箱 vs 基本类型
        example.test(100);          // int: 100 (基本类型优先)
        example.test(Integer.valueOf(100)); // Integer: 100
        
        // 4. 可变参数匹配
        example.test('A');          // char: A (基本类型优先于包装类)
        
        // 5. 模糊匹配需要强制转型
        example.test(null);         // 编译错误：模糊的方法调用
        
        // 明确指定类型
        example.test((String) null); // String: null
    }
}
```

### 2.3 静态分派的特点

```java
public class StaticDispatchCharacteristics {
    /*
    静态分派的特点：
    
    1. 编译期确定
       - 在编译阶段就确定了调用哪个方法
       - 编译后字节码中已经固定了方法符号引用
       
    2. 依赖静态类型
       - 根据变量声明时的类型（编译时类型）
       - 而不是实际指向的对象类型（运行时类型）
       
    3. 方法重载是典型应用
       - 同一个类中，方法名相同，参数不同
       - 编译时根据参数类型选择方法
       
    4. 静态方法、私有方法、final方法、构造方法
       - 这些方法都是静态分派的
       - 因为它们的调用目标在编译期就唯一确定
       
    5. 分派优先级规则：
       a) 精确匹配参数类型
       b) 自动类型转换（向上转型）
       c) 装箱/拆箱
       d) 可变参数
    */
    
    // 示例：静态方法调用也是静态分派
    static class StaticMethod {
        static void staticMethod() {
            System.out.println("StaticMethod.staticMethod");
        }
    }
    
    static class SubStaticMethod extends StaticMethod {
        static void staticMethod() {  // 这是隐藏（hide），不是重写（override）
            System.out.println("SubStaticMethod.staticMethod");
        }
    }
    
    public static void main(String[] args) {
        StaticMethod obj = new SubStaticMethod();
        obj.staticMethod();  // 输出: StaticMethod.staticMethod
        // 静态方法根据静态类型StaticMethod调用，不是动态分派
    }
}
```

## 三、动态分派（Dynamic Dispatch）

### 3.1 什么是动态分派？

动态分派发生在运行阶段，根据变量的**实际类型**（运行时类型）来确定调用的方法。

```java
public class DynamicDispatch {
    
    static abstract class Animal {
        abstract void speak();
        
        void eat() {
            System.out.println("Animal eats");
        }
    }
    
    static class Dog extends Animal {
        @Override
        void speak() {
            System.out.println("Dog barks");
        }
        
        @Override
        void eat() {
            System.out.println("Dog eats bones");
        }
    }
    
    static class Cat extends Animal {
        @Override
        void speak() {
            System.out.println("Cat meows");
        }
        
        void scratch() {
            System.out.println("Cat scratches");
        }
    }
    
    public static void main(String[] args) {
        // 动态分派演示
        Animal animal1 = new Dog();  // 编译时类型Animal，运行时类型Dog
        Animal animal2 = new Cat();  // 编译时类型Animal，运行时类型Cat
        
        // 动态分派：根据实际类型调用方法
        animal1.speak();  // 输出: Dog barks  (运行时决定)
        animal2.speak();  // 输出: Cat meows  (运行时决定)
        
        animal1.eat();    // 输出: Dog eats bones  (重写方法也动态分派)
        
        // 不能调用子类特有的方法
        // animal2.scratch();  // 编译错误：Animal类型没有scratch方法
        
        // 多态的实际应用
        Animal[] animals = {new Dog(), new Cat(), new Dog()};
        for (Animal animal : animals) {
            animal.speak();  // 运行时动态分派
        }
    }
}
```

### 3.2 方法重写与动态分派

方法重写（Override）是动态分派的典型应用：

```java
public class OverrideExample {
    
    static class Parent {
        public void normalMethod() {
            System.out.println("Parent.normalMethod");
        }
        
        public final void finalMethod() {
            System.out.println("Parent.finalMethod");
        }
        
        private void privateMethod() {
            System.out.println("Parent.privateMethod");
        }
        
        public static void staticMethod() {
            System.out.println("Parent.staticMethod");
        }
    }
    
    static class Child extends Parent {
        // 重写普通方法 - 动态分派
        @Override
        public void normalMethod() {
            System.out.println("Child.normalMethod");
        }
        
        // 不能重写final方法
        // public void finalMethod() {}  // 编译错误
        
        // 这不是重写，只是子类的新方法
        private void privateMethod() {
            System.out.println("Child.privateMethod");
        }
        
        // 这不是重写，是隐藏（hide）
        public static void staticMethod() {
            System.out.println("Child.staticMethod");
        }
        
        // 子类特有方法
        public void childMethod() {
            System.out.println("Child.childMethod");
        }
    }
    
    public static void main(String[] args) {
        Parent obj = new Child();
        
        // 动态分派：根据实际类型Child调用
        obj.normalMethod();      // 输出: Child.normalMethod
        
        // 静态分派：final方法在编译期绑定
        obj.finalMethod();       // 输出: Parent.finalMethod
        
        // 静态分派：私有方法在编译期绑定
        // obj.privateMethod();  // 编译错误：不可见
        
        // 静态分派：静态方法根据静态类型绑定
        obj.staticMethod();      // 输出: Parent.staticMethod
        
        // 需要强制转型才能调用子类特有方法
        // obj.childMethod();    // 编译错误
        
        if (obj instanceof Child) {
            ((Child) obj).childMethod();  // 输出: Child.childMethod
        }
    }
}
```

### 3.3 JVM 实现原理：虚方法表（vtable）

```java
public class VTableMechanism {
    /*
    JVM使用虚方法表实现动态分派：
    
    每个类都有一个虚方法表（vtable），包含：
    1. 所有非private、非final、非static的方法
    2. 从父类继承的可重写方法
    3. 本类定义的新方法
    
    虚方法表示例：
    
    Animal类的vtable:        Dog类的vtable:
    ┌─────────────────┐     ┌─────────────────┐
    │ speak()         │     │ speak()         │ ← 指向Dog.speak()
    │ eat()           │     │ eat()           │ ← 指向Dog.eat()
    │ toString()      │     │ toString()      │
    │ hashCode()      │     │ hashCode()      │
    │ ...             │     │ ...             │
    └─────────────────┘     └─────────────────┘
    
    调用过程：
    1. 获取对象的实际类型
    2. 查找该类型的虚方法表
    3. 根据方法签名找到对应槽位（slot）
    4. 调用槽位中指向的方法
    */
    
    // 查看字节码验证
    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.speak();  // invokevirtual指令
        
        // 使用javap查看字节码：
        // invokevirtual #4  // Method Animal.speak:()V
        // 运行时查找虚方法表
    }
    
    static class Animal {
        public void speak() { System.out.println("Animal"); }
    }
    
    static class Dog extends Animal {
        @Override
        public void speak() { System.out.println("Dog"); }
    }
}
```

### 3.4 动态分派的特点

```java
public class DynamicDispatchCharacteristics {
    /*
    动态分派的特点：
    
    1. 运行期确定
       - 在程序运行时才确定调用哪个方法
       - 字节码中使用invokevirtual指令
       
    2. 依赖实际类型
       - 根据对象实际指向的类型（运行时类型）
       - 而不是变量声明的类型（编译时类型）
       
    3. 方法重写是典型应用
       - 子类重写父类方法
       - 运行时根据对象实际类型调用
       
    4. 实现机制：虚方法表
       - 每个类维护一个虚方法表
       - 包含所有可重写方法的入口地址
       
    5. 性能考虑
       - 比静态分派慢（需要查表）
       - JIT会优化频繁调用的虚方法（内联缓存）
       
    6. 无法动态分派的方法
       - 静态方法：invokestatic指令
       - 私有方法、构造方法：invokespecial指令
       - final方法：虽然是invokevirtual，但可以静态绑定
    */
}
```

## 四、双重分派与访问者模式

### 4.1 单分派的局限性

```java
public class SingleDispatchLimitation {
    
    static class Printer {
        public void print(String str) {
            System.out.println("Print string: " + str);
        }
        
        public void print(Integer num) {
            System.out.println("Print integer: " + num);
        }
    }
    
    static class AdvancedPrinter extends Printer {
        @Override
        public void print(String str) {
            System.out.println("Advanced print string: " + str);
        }
        
        @Override
        public void print(Integer num) {
            System.out.println("Advanced print integer: " + num);
        }
    }
    
    public static void main(String[] args) {
        Object obj1 = "Hello";
        Object obj2 = 123;
        
        Printer printer = new AdvancedPrinter();
        
        // 问题：Java只支持单分派
        // 这里只能根据printer的实际类型动态分派
        // 但不能根据参数类型动态分派
        
        // printer.print(obj1);  // 编译错误：找不到合适的方法
        // printer.print(obj2);  // 编译错误：找不到合适的方法
        
        // 需要强制转型
        printer.print((String) obj1);  // Advanced print string: Hello
        printer.print((Integer) obj2); // Advanced print integer: 123
    }
}
```

### 4.2 访问者模式实现双重分派

```java
import java.util.Arrays;
import java.util.List;

public class DoubleDispatchExample {
    
    // 访问者接口
    interface Visitor {
        void visit(Circle circle);
        void visit(Rectangle rectangle);
        void visit(Triangle triangle);
    }
    
    // 具体访问者
    static class DrawVisitor implements Visitor {
        @Override
        public void visit(Circle circle) {
            System.out.println("Drawing Circle with radius: " + circle.radius);
        }
        
        @Override
        public void visit(Rectangle rectangle) {
            System.out.println("Drawing Rectangle: " + rectangle.width + "x" + rectangle.height);
        }
        
        @Override
        public void visit(Triangle triangle) {
            System.out.println("Drawing Triangle with base: " + triangle.base + ", height: " + triangle.height);
        }
    }
    
    static class AreaVisitor implements Visitor {
        @Override
        public void visit(Circle circle) {
            double area = Math.PI * circle.radius * circle.radius;
            System.out.println("Circle area: " + area);
        }
        
        @Override
        public void visit(Rectangle rectangle) {
            double area = rectangle.width * rectangle.height;
            System.out.println("Rectangle area: " + area);
        }
        
        @Override
        public void visit(Triangle triangle) {
            double area = 0.5 * triangle.base * triangle.height;
            System.out.println("Triangle area: " + area);
        }
    }
    
    // 元素接口
    interface Shape {
        void accept(Visitor visitor);
    }
    
    // 具体元素
    static class Circle implements Shape {
        double radius;
        
        Circle(double radius) {
            this.radius = radius;
        }
        
        @Override
        public void accept(Visitor visitor) {
            visitor.visit(this);  // 第一次分派：根据visitor类型
            // 第二次分派：visitor.visit(this)中的this是具体类型
        }
    }
    
    static class Rectangle implements Shape {
        double width, height;
        
        Rectangle(double width, double height) {
            this.width = width;
            this.height = height;
        }
        
        @Override
        public void accept(Visitor visitor) {
            visitor.visit(this);
        }
    }
    
    static class Triangle implements Shape {
        double base, height;
        
        Triangle(double base, double height) {
            this.base = base;
            this.height = height;
        }
        
        @Override
        public void accept(Visitor visitor) {
            visitor.visit(this);
        }
    }
    
    public static void main(String[] args) {
        List<Shape> shapes = Arrays.asList(
            new Circle(5),
            new Rectangle(3, 4),
            new Triangle(6, 8)
        );
        
        Visitor drawVisitor = new DrawVisitor();
        Visitor areaVisitor = new AreaVisitor();
        
        System.out.println("=== Drawing Shapes ===");
        for (Shape shape : shapes) {
            shape.accept(drawVisitor);  // 双重分派
        }
        
        System.out.println("\n=== Calculating Areas ===");
        for (Shape shape : shapes) {
            shape.accept(areaVisitor);  // 双重分派
        }
    }
}
```

## 五、分派机制的实际应用

### 5.1 设计模式中的应用

```java
public class DispatchInDesignPatterns {
    
    // 1. 模板方法模式 - 动态分派
    static abstract class Game {
        // 模板方法
        public final void play() {  // final方法，静态分派
            initialize();
            startPlay();
            endPlay();
        }
        
        abstract void initialize();    // 子类重写 - 动态分派
        abstract void startPlay();     // 子类重写 - 动态分派
        abstract void endPlay();       // 子类重写 - 动态分派
    }
    
    static class Cricket extends Game {
        @Override
        void initialize() {
            System.out.println("Cricket Game Initialized");
        }
        
        @Override
        void startPlay() {
            System.out.println("Cricket Game Started");
        }
        
        @Override
        void endPlay() {
            System.out.println("Cricket Game Finished");
        }
    }
    
    // 2. 策略模式 - 结合静态和动态分派
    interface PaymentStrategy {
        void pay(int amount);
    }
    
    static class CreditCardPayment implements PaymentStrategy {
        @Override
        public void pay(int amount) {
            System.out.println("Paid " + amount + " using Credit Card");
        }
    }
    
    static class PayPalPayment implements PaymentStrategy {
        @Override
        public void pay(int amount) {
            System.out.println("Paid " + amount + " using PayPal");
        }
    }
    
    static class ShoppingCart {
        private PaymentStrategy paymentStrategy;
        
        // 静态分派：根据参数类型选择方法
        public void setPaymentStrategy(PaymentStrategy strategy) {
            this.paymentStrategy = strategy;
        }
        
        public void checkout(int amount) {
            // 动态分派：根据实际类型调用pay方法
            paymentStrategy.pay(amount);
        }
    }
    
    public static void main(String[] args) {
        // 模板方法模式示例
        Game game = new Cricket();
        game.play();  // 动态分派调用子类方法
        
        // 策略模式示例
        ShoppingCart cart = new ShoppingCart();
        
        cart.setPaymentStrategy(new CreditCardPayment());
        cart.checkout(100);  // 动态分派
        
        cart.setPaymentStrategy(new PayPalPayment());
        cart.checkout(200);  // 动态分派
    }
}
```

### 5.2 性能优化考虑

```java
public class DispatchPerformance {
    
    static interface Operation {
        int execute(int a, int b);
    }
    
    static class Add implements Operation {
        @Override
        public int execute(int a, int b) {
            return a + b;
        }
    }
    
    static class Multiply implements Operation {
        @Override
        public int execute(int a, int b) {
            return a * b;
        }
    }
    
    // 测试动态分派的性能
    public static long testDynamicDispatch(Operation op, int iterations) {
        long start = System.nanoTime();
        int result = 0;
        for (int i = 0; i < iterations; i++) {
            result += op.execute(i, i + 1);  // 动态分派
        }
        return System.nanoTime() - start;
    }
    
    // 测试静态分派的性能
    public static long testStaticDispatch(Add add, int iterations) {
        long start = System.nanoTime();
        int result = 0;
        for (int i = 0; i < iterations; i++) {
            result += add.execute(i, i + 1);  // 静态分派（可内联优化）
        }
        return System.nanoTime() - start;
    }
    
    public static void main(String[] args) {
        int iterations = 100_000_000;
        
        Operation dynamicOp = new Add();
        Add staticOp = new Add();
        
        long dynamicTime = testDynamicDispatch(dynamicOp, iterations);
        long staticTime = testStaticDispatch(staticOp, iterations);
        
        System.out.println("动态分派耗时: " + dynamicTime + " ns");
        System.out.println("静态分派耗时: " + staticTime + " ns");
        System.out.println("性能差异: " + (dynamicTime - staticTime) + " ns");
        
        /*
        性能差异原因：
        1. 动态分派需要查虚方法表
        2. 静态分派可被JIT内联优化
        3. 对于final方法，JVM可以静态绑定，性能接近静态分派
        
        优化建议：
        1. 将高频调用的方法标记为final（如果允许）
        2. 使用内联缓存优化（JIT自动完成）
        3. 避免在性能关键路径上过度使用多态
        */
    }
}
```

## 六、总结对比

```java
public class DispatchSummary {
    /*
    ==================== 静态分派 vs 动态分派 ====================
    
    维度          | 静态分派                     | 动态分派
    -------------|------------------------------|---------------------------
    发生时间      | 编译期                       | 运行期
    决定因素      | 变量的声明类型（静态类型）     | 对象的实际类型（运行时类型）
    典型应用      | 方法重载（Overload）         | 方法重写（Override）
    JVM指令      | invokestatic, invokespecial  | invokevirtual, invokeinterface
    实现机制      | 直接方法调用                  | 虚方法表（vtable）
    性能         | 高（可内联优化）              | 较低（需要查表）
    灵活性       | 低                           | 高（支持多态）
    确定性       | 确定（编译时已知）            | 不确定（运行时决定）
    
    无法动态分派的方法：
    1. 静态方法 - 属于类，不属于对象
    2. 私有方法 - 不可被重写
    3. final方法 - 不可被重写（但用invokevirtual指令）
    4. 构造方法 - 特殊的方法
    
    最佳实践：
    1. 对于性能关键的代码，考虑使用final方法
    2. 合理使用重载和重写，明确设计意图
    3. 理解分派机制有助于编写高效、清晰的代码
    4. 在设计API时，考虑用户如何使用分派特性
    */
    
    // 实际代码示例总结
    public void demonstrate() {
        // 静态分派示例
        Math.max(1, 2);          // 根据参数类型选择重载方法
        
        // 动态分派示例
        List<String> list = new ArrayList<>();
        list.add("item");        // ArrayList的add方法（动态分派）
        
        // 混合使用
        Object obj = "Hello";
        System.out.println(obj); // 1. 选择println(Object)重载（静态）
                                 // 2. 调用String.toString()（动态）
    }
}
```

## 七、高级话题：invokedynamic

Java 7 引入的 `invokedynamic` 指令为动态语言支持提供了新的分派机制：

```java
import java.lang.invoke.*;

public class InvokeDynamicExample {
    
    // 使用MethodHandle实现动态分派
    public static void methodHandleExample() throws Throwable {
        MethodHandles.Lookup lookup = MethodHandles.lookup();
        
        // 获取方法句柄
        MethodHandle mh = lookup.findVirtual(String.class, 
                                           "length", 
                                           MethodType.methodType(int.class));
        
        String str = "Hello";
        // 动态调用
        int length = (int) mh.invokeExact(str);
        System.out.println("Length: " + length);
    }
    
    // Lambda表达式使用invokedynamic
    public static void lambdaExample() {
        // Lambda表达式在编译时生成invokedynamic指令
        Runnable r = () -> System.out.println("Hello Lambda");
        r.run();  // 运行时动态绑定实现
        
        // 相当于：
        // invokedynamic #0:run()Ljava/lang/Runnable;
    }
    
    public static void main(String[] args) throws Throwable {
        methodHandleExample();
        lambdaExample();
    }
}
```

**方法调用指令**

| 指令              | 用途         | 特点                       | 分派类型 |
| :---------------- | :----------- | :------------------------- | :------- |
| `invokestatic`    | 调用静态方法 | 类级别，不依赖对象         | 静态分派 |
| `invokespecial`   | 调用特殊方法 | 构造器、私有方法、父类方法 | 静态分派 |
| `invokevirtual`   | 调用虚方法   | 普通实例方法，支持多态     | 动态分派 |
| `invokeinterface` | 调用接口方法 | 接口方法调用               | 动态分派 |
| `invokedynamic`   | 动态调用     | 动态语言支持、Lambda       | 动态绑定 |