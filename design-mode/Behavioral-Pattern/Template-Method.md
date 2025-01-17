### 一、模版方法模式的定义

模板方法模式是类的行为模式。准备一个抽象类，将部分逻辑以具体方法以及具体构造函数的形式实现，然后声明一些抽象方法来迫使子类实现剩余的逻辑。不同的子类可以以不同的方式实现这些抽象方法，从而对剩余的逻辑有不同的实现。这就是模板方法模式的用意。

### 二、模版方法模式的结构

模板方法模式是所有模式中最为常见的几个模式之一，是基于继承的代码复用的基本技术。

模板方法模式需要开发抽象类和具体子类的设计师之间的协作。一个设计师负责给出一个算法的轮廓和骨架，另一些设计师则负责给出这个算法的各个逻辑步骤。代表这些具体逻辑步骤的方法称做基本方法\(primitive method\)；而将这些基本方法汇总起来的方法叫做模板方法\(template method\)，这个设计模式的名字就是从此而来。

模板方法所代表的行为称为顶级行为，其逻辑称为顶级逻辑。模板方法模式的静态结构图如下所示：

![](https://upload-images.jianshu.io/upload_images/3985563-05618f06dfa3ab58.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
这里涉及到两个角色：

**抽象模板\(Abstract Template\)角色有如下责任：**

■　　定义了一个或多个抽象操作，以便让子类实现。这些抽象操作叫做基本操作，它们是一个顶级逻辑的组成步骤。

■　　定义并实现了一个模板方法。这个模板方法一般是一个具体方法，它给出了一个顶级逻辑的骨架，而逻辑的组成步骤在相应的抽象操作中，推迟到子类实现。顶级逻辑也有可能调用一些具体方法。

**具体模板\(Concrete Template\)角色又如下责任：**

■　　实现父类所定义的一个或多个抽象方法，它们是一个顶级逻辑的组成步骤。

■　　每一个抽象模板角色都可以有任意多个具体模板角色与之对应，而每一个具体模板角色都可以给出这些抽象方法（也就是顶级逻辑的组成步骤）的不同实现，从而使得顶级逻辑的实现各不相同。

**抽象模板角色类，abstractMethod\(\)、hookMethod\(\)等基本方法是顶级逻辑的组成步骤，这个顶级逻辑由templateMethod\(\)方法代表。**

```java
public abstract class AbstractTemplate {
    /**
     * 模板方法
     */
    public void templateMethod(){
        //调用基本方法
        abstractMethod();
        hookMethod();
        concreteMethod();
    }
    /**
     * 基本方法的声明（由子类实现）
     */
    protected abstract void abstractMethod();
    /**
     * 基本方法(空方法)
     */
    protected void hookMethod(){}
    /**
     * 基本方法（已经实现）
     */
    private final void concreteMethod(){
        //业务相关的代码
    }
}
```

**具体模板角色类，实现了父类所声明的基本方法，abstractMethod\(\)方法所代表的就是强制子类实现的剩余逻辑，而hookMethod\(\)方法是可选择实现的逻辑，不是必须实现的。**

```java
public class ConcreteTemplate extends AbstractTemplate{
    //基本方法的实现
    @Override
    public void abstractMethod() {
        //业务相关的代码
    }
    //重写父类的方法
    @Override
    public void hookMethod() {
        //业务相关的代码
    }
}
```

模板模式的关键是：**子类可以置换掉父类的可变部分，但是子类却不可以改变模板方法所代表的顶级逻辑。**

每当定义一个新的子类时，不要按照控制流程的思路去想，而应当按照“责任”的思路去想。换言之，应当考虑哪些操作是必须置换掉的，哪些操作是可以置换掉的，以及哪些操作是不可以置换掉的。使用模板模式可以使这些责任变得清晰。

### 三、模板方法模式中的方法

模板方法中的方法可以分为两大类：模板方法和基本方法。

##### 模板方法

一个模板方法是定义在抽象类中的，把基本操作方法组合在一起形成一个总算法或一个总行为的方法。

一个抽象类可以有任意多个模板方法，而不限于一个。每一个模板方法都可以调用任意多个具体方法。

##### 基本方法

基本方法又可以分为三种：抽象方法\(Abstract Method\)、具体方法\(Concrete Method\)和钩子方法\(Hook Method\)。

●　　抽象方法：一个抽象方法由抽象类声明，由具体子类实现。在Java语言里抽象方法以abstract关键字标示。

●　　具体方法：一个具体方法由抽象类声明并实现，而子类并不实现或置换。

●　　钩子方法：一个钩子方法由抽象类声明并实现，而子类会加以扩展。通常抽象类给出的实现是一个空实现，作为方法的默认实现。

在上面的例子中，AbstractTemplate是一个抽象类，它带有三个方法。其中abstractMethod\(\)是一个抽象方法，它由抽象类声明为抽象方法，并由子类实现；hookMethod\(\)是一个钩子方法，它由抽象类声明并提供默认实现，并且由子类置换掉。concreteMethod\(\)是一个具体方法，它由抽象类声明并实现。

##### 默认钩子方法

一个钩子方法常常由抽象类给出一个空实现作为此方法的默认实现。这种空的钩子方法叫做“Do Nothing Hook”。具体模版类中可以选择是否重写钩子方法，通常重写钩子方法是为了对模版方法中的步骤进行控制，判断钩子方法中的状态，是否进行下一步操作。

### 四、模版方法的具体实例

考虑一个计算存款利息的例子。假设系统需要支持两种存款账号，即货币市场\(Money Market\)账号和定期存款\(Certificate of Deposite\)账号。这两种账号的存款利息是不同的，因此，在计算一个存户的存款利息额时，必须区分两种不同的账号类型。

这个系统的总行为应当是计算出利息，这也就决定了作为一个模板方法模式的顶级逻辑应当是利息计算。由于利息计算涉及到两个步骤：一个基本方法给出账号种类，另一个基本方法给出利息百分比。这两个基本方法构成具体逻辑，因为账号的类型不同，所以具体逻辑会有所不同。

显然，系统需要一个抽象角色给出顶级行为的实现，而将两个作为细节步骤的基本方法留给具体子类实现。由于需要考虑的账号有两种：一是货币市场账号，二是定期存款账号。系统的类结构如下图所示。

![](https://upload-images.jianshu.io/upload_images/3985563-fd074443f7382c2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

  


  
**抽象模板角色类**

```java
public abstract class Account {
    /**
     * 模板方法，计算利息数额
     * @return    返回利息数额
     */
    public final double calculateInterest(){
        double interestRate = doCalculateInterestRate();
        String accountType = doCalculateAccountType();
        double amount = calculateAmount(accountType);
        return amount * interestRate;
    }
    /**
     * 基本方法留给子类实现
     */
    protected abstract String doCalculateAccountType();
    /**
     * 基本方法留给子类实现
     */
    protected abstract double doCalculateInterestRate();
    /**
     * 基本方法，已经实现
     */
    private double calculateAmount(String accountType){
        /**
         * 省略相关的业务逻辑
         */
        return 7243.00;
    }
}
```

**具体模板角色类**

```java
public class MoneyMarketAccount extends Account {

    @Override
    protected String doCalculateAccountType() {

        return "Money Market";
    }

    @Override
    protected double doCalculateInterestRate() {

        return 0.045;
    }

}
```

```java
public class CDAccount extends Account {

    @Override
    protected String doCalculateAccountType() {
        return "Certificate of Deposite";
    }

    @Override
    protected double doCalculateInterestRate() {
        return 0.06;
    }

}
```

**客户端类**

```java
public class Client {

    public static void main(String[] args) {
        Account account = new MoneyMarketAccount();
        System.out.println("货币市场账号的利息数额为：" + account.calculateInterest());
        account = new CDAccount();
        System.out.println("定期账号的利息数额为：" + account.calculateInterest());
    }

}
```

### 五、模板方法模式效果与适用场景

模板方法模式是基于继承的代码复用技术，它体现了面向对象的诸多重要思想，是一种使用较为频繁的模式。模板方法模式广泛应用于框架设计中，以确保通过父类来控制处理流程的逻辑顺序（如框架的初始化，测试流程的设置等）。

在以下情况下可以考虑使用模板方法模式：

\(1\) 对一些复杂的算法进行分割，将其算法中固定不变的部分设计为模板方法和父类具体方法，而一些可以改变的细节由其子类来实现。即：一次性实现一个算法的不变部分，并将可变的行为留给子类来实现。

\(2\) 各子类中公共的行为应被提取出来并集中到一个公共父类中以避免代码重复。

\(3\) 需要通过子类来决定父类算法中某个步骤是否执行，实现子类对父类的反向控制。

### 六、模版方法模式的优缺点

##### 优点

\(1\) 在父类中形式化地定义一个算法，而由它的子类来实现细节的处理，在子类实现详细的处理算法时并不会改变算法中步骤的执行次序。

\(2\) 模板方法模式是一种代码复用技术，它在类库设计中尤为重要，它提取了类库中的公共行为，将公共行为放在父类中，而通过其子类来实现不同的行为，它鼓励我们恰当使用继承来实现代码复用。

\(3\) 可实现一种反向控制结构，通过子类覆盖父类的钩子方法来决定某一特定步骤是否需要执行。

\(4\) 在模板方法模式中可以通过子类来覆盖父类的基本方法，不同的子类可以提供基本方法的不同实现，更换和增加新的子类很方便，符合单一职责原则和开闭原则。

##### 缺点

需要为每一个基本方法的不同实现提供一个子类，如果父类中可变的基本方法太多，将会导致类的个数增加，系统更加庞大，设计也更加抽象，此时，可结合桥接模式来进行设计。

