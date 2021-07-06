# java 抽象类&接口

## 抽象类

抽象类体现了数据抽象的思想，是实现多态的一种机制。抽象类定义了一组抽象的方法，至于这组抽象方法的具体表现形式由子类来继承实现。抽象类就是用来继承的，否则它就没有存在的任何意义。

##### 特点：

1.抽象类可以有构造方法，但不能创建对象，只能继承

2.抽象类可以没有抽象方法

3.抽象方法不可以用static.final修饰 访问权限不能用private修饰

4.抽象类的子类必须实现抽象类所有的抽象方法，否则子类也必须使抽象类

```java
public abstract class AbstractDemo {
    public String strOne;

    //静态常量成员变量
    private static final String str = "2";

    //构造方法
    public AbstractDemo() {
    }

    public AbstractDemo(String strOne) {
        this.strOne = strOne;
    }

    /**
     * 抽象方法
     */
    public abstract void abstractMethod();

    public void commonMethod(){
        System.out.println("普通方法");
    }
}
```

## 接口

java里只支持单继承，那么接口的出现使得java语言就可以实现多继承（接口），在面向对象的编程，如果要提高程序的复用率，增加程序的可维护性，可扩展性就必须是面向接口的编程。接口只是一种形式，就好像一纸契约，自身不能做任何事情。但只要某个类实现了这个接口，就必须按照这纸契约来办事：接口里提到的方法必须全部实现，少一个都不行。

##### 特点：

1. 方法:JDK7及其以下版本JDK只能包含公开且抽象的方法（默认由public abstract修饰），而JDK8开始可以包含default、static修饰的非抽象方法，JDK 9：私有方法
2. 通过implements让子类实现
3. 接口是一个特殊的抽象类
4. 接口和接口可以多继承 ，接口和类之间可以多实现
5. 接口突破了java的单继承
6. 接口是对外暴露是一套开发标准
7. 接口提高的程序的功能扩展性，降低了耦合性（耦合性越低，性能越强）
8. 成员变量:只能包含公开静态常量(默认public static final)
9. 接口不允许有构造方法

```java
public interface InterfaceDemo {
    /**
     * 成员变量:只能包含公开静态常量(默认public static final)
     */
    String strOne = null;
    /**
     * JDK8可以增加static方法
     */
    static void staticMethod(){
        System.out.println("ss");
    }
    /**
     * JDK8可以增加default方法
     */
    default void defaultMethod(){
        System.out.println("sss");
    }

    /**
     * 接口方法
     */
    void interfaceMethhod();

}

```

> 抽象类的设计目的，是代码复用。当不同的类具有某些相同的行为(记为行为集合A)，且其中一部分行为的实现方式一致时（A的非真子集，记为B），可以让这些类都派生于一个抽象类。在这个抽象类中实现了B，避免让所有的子类来实现B，这就达到了代码复用的目的。而A减B的部分，留给各个子类自己实现。正是因为A-B在这里没有实现，所以抽象类不允许实例化出来（否则当调用到A-B时，无法执行）。
>
> 接口的设计目的，是对类的行为进行约束（更准确的说是一种“有”约束，因为接口不能规定类不可以有什么行为），也就是提供一种机制，可以强制要求不同的类具有相同的行为。它只约束了行为的有无，但不对如何实现行为进行限制。对“接口为何是约束”的理解，我觉得配合泛型使用效果更佳。
>
> --摘自知乎