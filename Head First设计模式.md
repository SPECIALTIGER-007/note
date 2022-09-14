## 1、策略模式

### 定义

策略模式（Strategy Pattern）属于对象的行为模式。其用意是针对一组算法，将每一个算法封装到具有共同接口的独立的类中，从而使得它们可以相互替换。策略模式使得算法可以在不影响到客户端的情况下发生变化。

### 内容

**策略模式**主要由这三个角色组成，环境角色(Context)、抽象策略角色(Strategy)和具体策略角色(ConcreteStrategy)。

- 环境角色(Context)：持有一个策略类的引用，提供给客户端使用。
- 抽象策略角色(Strategy)：这是一个抽象角色，通常由一个接口或抽象类实现。此角色给出所有的具体策略类所需的接口。
- 具体策略角色(ConcreteStrategy)：包装了相关的算法或行为。

### 个人理解

![在这里插入图片描述](https://img-blog.csdnimg.cn/20181111152930787.png)

相当于在一个类中，**定义一些接口作为成员属性**，再通过这些接口的具体实现类实现不同的方法功能。（创建该类的时候，可以使用多态赋予成员属性不同的实现）

### 优缺点

#### 优点

- 策略模式的Strategy类层次为Context定义了一系列的可供重用的算法或行为。继承有助于析取出这些算法中的公共功能
- 简化了单元测试，每个算法都有自己的类，可以通过自己的接口单独测试
- 消除条件语句并且扩展性良好

#### 缺点

- 所有策略类都需要对外暴露
- 策略类会不断增多

## 2、观察者模式

### 定义

观察者模式，也被称为发布订阅模式，是一种行为型设计模式，也是在实际的开发中用得比较多的一种模式，当对象间存在一对多关系时，就可以使用观察者模式。定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象（观察者对象）都得到通知并被自动更新。

![在这里插入图片描述](https://img-blog.csdnimg.cn/debb8de6e405440b92eaf7861a57c090.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAWXNtaW5nODg=,size_20,color_FFFFFF,t_70,g_se,x_16)

### 个人理解

**在Subject类中保存一个Observer集合**，提供注册，删除等功能。当Subject调用更新属性的方法时，通知集合里的Observer，调用Observer的update()方法，在Observer的update方法中更新数据或者展示数据。

### 优缺点

#### 优点

 1、观察者和被观察者是抽象耦合的。它是将观察者和被观察者代码解耦 2、建立一套触发机制。

#### 缺点

1、如果一个被观察者对象有很多的直接和间接的观察者的话，将所有的观察者都通知到会花费很多时间。
2、如果在观察者和观察目标之间有循环依赖的话，观察目标会触发它们之间进行循环调用，可能导致系统崩溃。
3、观察者模式没有相应的机制让观察者知道所观察的目标对象是怎么发生变化的，而仅仅只是知道观察目标发生了变化。

观察者模式的应用场景非常广泛，小到代码层面的解耦，大到架构层面的系统解耦，再或者一些产品的设计思路，都有这种模式的影子，比如，邮件订阅、RSS Feeds，本质上都是观察者模式。

## 3、装饰者模式

### 定义

装饰者（Decorator）模式的定义：指在不改变现有对象结构的情况下，动态地给该对象增加一些职责（即增加其额外功能）的模式，它属于对象结构型模式。

### 个人理解

![image-20220908190127308](E:\笔记\Head First设计模式.assets\image-20220908190127308.png)

如上图所示,最里面的DarkRoast类是被装饰的组件,外面两层都是具体的装饰者,通过层层包装的模式,为组件添加功能

![image-20220908191015088](E:\笔记\Head First设计模式.assets\image-20220908191015088.png)

如上图,Beverage是抽象的组件类,HouseBlend等是具体的组件类,CondimentDecorator是抽象的装饰者类,Milk等是具体的装饰者类.

装饰者将组件类作为构造器参数传入,然后需要几个装饰者类就再将装饰者类作为构造器参数传入.  例如:

```java
Beverage beverage2 = new DarkRoast();
beverage2 = new Mocha(beverage2);
beverage2 = new Mocha(beverage2);
beverage2 = new Whip(beverage2);
System.out.println(beverage2.getDescription() + "$" + beverage2.cost());
```

### jdk源码的使用

java.io包中,就使用了装饰者模式

![image-20220908191845789](E:\笔记\Head First设计模式.assets\image-20220908191845789.png)

FileInputStream是具体的组件,BufferedInputStream和ZipInputStream是具体的装饰者

使用样例:

LowerCaseInputStream是自定义的装饰者类

```java
InputStream in = new LowerCaseInputStream(
    new BufferedInputStream(
        new FileInputStream("src/test.txt")
    )
);
while ((c = in.read()) >= 0) {
    System.out.print((char) c);
}
in.close();
```

### 特点

1. 比继承灵活,在不改变现有对象的情况下,动态的给一个对象扩展功能
2. 使用不同的装饰类和这些装饰类的排列组合,可以实现不同的效果
3. 装饰者模式符合开闭原则 (对扩展开放,对修改关闭)

### 缺点

1. 会产生大量子类,过度使用会增加复杂性
2. 如果代码依赖于具体组件类型,装饰者会破坏这个代码

## 4、工厂模式

### 工厂方法模式

#### 定义

工厂方法模式定义了一个创建对象的接口，但由子类决定要实例化哪个类。工厂方法让类把实例化推迟到子类。

### 抽象工厂模式

#### 定义

抽象工厂模式提供一个接口来创建相关或依赖对象的家族，而不需要指定具体类。

抽象工厂模式经常使用工厂方法模式来创建具体的产品

### 个人理解

#### 工厂方法模式

![image-20220909231024635](E:\笔记\Head First设计模式.assets\image-20220909231024635.png)

上图是工厂方法模式的类图

PizzaStore是抽象类，而ChicagoPizzaStor e和NYPizzaStore是它的具体实现类，抽象类中有一个抽象方法createPizza()，在子类中实现这些方法，每一个子类通过传入的参数生成不同类型的Pizza，下图是PizzaStore的orderPizza方法的默认实现，在此方法中，调用子类具体实现的createPizza()。

```java
public Pizza orderPizza(String type) {
    Pizza pizza = createPizza(type);
    System.out.println("--- Making a " + pizza.getName() + " ---");
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```

```java
Pizza createPizza(String item) {
    if (item.equals("cheese")) {
        return new ChicagoStyleCheesePizza();
    } else if (item.equals("veggie")) {
        return new ChicagoStyleVeggiePizza();
    } else if (item.equals("clam")) {
        return new ChicagoStyleClamPizza();
    } else if (item.equals("pepperoni")) {
        return new ChicagoStylePepperoniPizza();
    } else return null;
}
```

下面是测试类

```java
PizzaStore nyStore = new NYPizzaStore();
PizzaStore chicagoStore = new ChicagoPizzaStore();

Pizza pizza = nyStore.orderPizza("cheese");
System.out.println("Ethan ordered a " + pizza.getName() + "\n");

pizza = chicagoStore.orderPizza("cheese");
System.out.println("Joel ordered a " + pizza.getName() + "\n");
```

#### 工厂方法模式总结

整体思想就是使用子类来创建，这样客户只需要知道所用的抽象类型，子类操心具体的类型，客户将从具体类型解耦。

工厂方法模式将直接对象构造调用(使用new操作符)替换为对特殊工厂方法的调用。对象仍然是通过new操作符创建的，但它是从工厂方法内部调用的

![image-20220909235659740](E:\笔记\Head First设计模式.assets\image-20220909235659740.png)

如上图，Creator是一个类，包含操纵产品的所有方法的实现，除了工厂方法，而ConcreteCreator实现了工厂方法，这是生产实际产品的方法

#### 抽象工厂模式

![image-20220910001422148](E:\笔记\Head First设计模式.assets\image-20220910001422148.png)

具体的工厂实现不同的产品家族，客户使用其中一个工厂创建产品，绝对不用实例化产品对象。每一个具体的工厂能生产一整套的产品。

### 简单工厂模式，工厂方法模式和抽象工厂模式的区别

1. 简单工厂：唯一工厂类，一个产品抽象类，工厂类的创建方法依据入参判断并创建具体产品对象。
2. 工厂方法：多个工厂类，一个产品抽象类，利用多态创建不同的产品对象，避免了大量的if-else判断。
3. 抽象工厂：多个工厂类，多个产品抽象类，产品子类分组，同一个工厂实现类创建同组中的不同产品，减少了工厂子类的数量。

## 5、单例模式

### 定义

单例模式确保一个类只有一个实例，并提供一个全局访问点 

### 简单模式

```java
public class Singleton {
    private Singleton() {
    }

    private static Singleton uniqueInstance;

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

缺点是 多线程时，当多个线程同时进入if语句后，会产生多个实例

### 多线程模式下有三种方法

#### synchronized方法

直接在方法前加入关键词 synchronized，缺点是性能很差

#### 饿汉模式

在创建类时就创建好单例

#### 双重检查加锁

双重检查

```java
public class Singleton {
    private Singleton() {
    }

    private volatile static Singleton uniqueInstance;

    public static Singleton getInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) {
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

比单纯的使用synchronized方法开销小

### 特点

- 1、单例类只能有一个实例。
- 2、单例类必须自己创建自己的唯一实例。
- 3、单例类必须给所有其他对象提供这一实例。

## 6、 命令模式
