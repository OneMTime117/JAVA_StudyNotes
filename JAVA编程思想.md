# **JAVA**编程思想

## 控制执行流程

**1、void方法也可以使用retrun跳出方法；void方法默认最后有一个隐性retrun，表示跳出结束方法**

## 初始化和清理

**1、创建对象时，需要调用类的构造器（构造方法）来初始化对象**

**2、为什么不能以返回值区分重载方法：有时不一定需要方法的返回值，因此java就不能判断区分重载方法**

**3、this关键字，用于在当前对象中，获取当前对象，用于区分与对象名相同的其他参数与方法名**

**4、static方法中不能使用this关键字，因为static方法可以直接通过该类来调用；而this代表的是该类的实例，即对象**

**5、由于java垃圾回收机制本身有开销，因此垃圾回收只有当内存资源不足时，才会进行**

**6、finalize（）方法调用并不会使对象被垃圾回收，只是表面该对象可以被回收（只有当垃圾回收执行时，才会被自动销毁）；java自动垃圾回收机制会将不需要的对象，自动隐性的调用该方法，让java知道该对象需要被销毁**

**7、finalize（）的显性使用：用于java中调用本地方法（其他语言C，C++）时，无法被java垃圾回收机制自动获取其是否需要保留，此时就需要自己手动在本地方法中调用finalize（）**

**8、垃圾回收器工作原理：**

停止-复制：将程序停止工作，再将所有存活的对象复制到另一个堆空间，通过这样来减少对象在堆中的排序过程的复杂性（直接在垃圾清理时完成），增加java分配动态内存空间的速度（当对象在堆中连续排列时，堆的内存分配会变的很快）

标记-清理：将程序停止工作，遍历所有对象，将存活对象标记，之后再将没有标记的对象销毁；速度慢，需要重新整理对象的排序，但垃圾少时，速度很快

垃圾回收器根据情况不同，自动切换两种工作方式：

当内存垃圾很多时，执行停止-复制；当垃圾很少时，执行标记-清理

**9、成员变量都会自动初始化；局部变量需要手动初始化**

**10、类初始化顺序：静态成员变量-----成员变量-----构造方法**

**所有成员变量在声明时，会按照代码顺序执行初始化（没有赋值的，进行默认初始化）**

**成员变量可以在构造方法中进行初始化，但此时其实已经执行了一次默认初始化，因此可以看作构造方法只是对成员变量进行一次赋值**

**11、成员变量初始化值：数字为0；布尔为false；字符为‘’；对象为null/**

**12、成员变量进行初始化时，可以使用{}来自己初始化成员变量；{}发生在构造方法前，一般用于执行多条初始化语句（如初始化map集合）；可以被static修饰，初始化静态成员变量，发生在成员变量初始化前**

**并且{}执行也会根据代码顺序依次执行**

**13、静态成员变量只在首次生成该类（实例化、或者直接调用该类的静态方法、静态成员变量）时，被初始化。之后再被使用时，不会执行静态成员变量的初始化，因此静态成员变量不能在构造方法中进行初始化**

**14、static方法中只能直接调用static方法（不能直接调用同类的普通方法），可以通过创建对象，来调用该对象的普通方法**

15、类的初始化是通过JVM装载类来完成的。对于静态变量的初始化,JVM隐式的实现了线程安全；而对于普通成员变量，在多线程下，它们各属于不同的对象，因此不存在线程安全问题

**16、java SE5新特性：**

1、可变参数列表，（ int ... b）  其实为一个数组，但提供参数时，是一个一个的int数据，编译器会将这些数据填充到一个数组中，在方法中作为实参 

2、枚举，若switch的key为一个枚举类型的变量，那么case 中的条件都应为该枚举类型的常量集中的值

## 访问权限控制

**1、关键字import用于引用包，关键字package用于声明该类在哪个包中**（java IDE会根据开发者右键创建class的位置，来自动添加package声明；当右键项目创建时，创建添加一个默认包，但不会通过package声明，因为该类被放在src根目录下，不会创建目录；但是默认与其他在根目录下的类在同一默认包中）

**2、权限修饰关键字：**

public  ：公开   

protected  ：受保护，同包或被继承时可以访问    

默认（没有修饰）：在同包中可以访问

private： 私有，只能在类中访问（一般通过在类中创建public方法来访问）

**3、private来修饰构造方法，并提供一个public方法来执行返回构造方法（单例模式）**

## 复用类

**1、java类的代码复用是通过两种方式：组合、继承**

组合：在一个类中创建另一个类的对象，从而调用其功能代码；继承：直接复用类的整个形式，并暴露父类的代码

**2、继承时，导出类（子类）构造器调用时必须调用基类（父类）构造器，无参构造器会自动调用；有参构造器需要手动调用，通过super（参数列表）**

**3、继承时，基类构造器调用时，是在导出类构造器之前；即从上到下（父类构造方法先执行）**

4、**java在继承时，方法会被继承，不会被屏蔽；可以对父类方法进行重载或重写，为了减少开发者分不清该方法是否为重写时，可以通过对方法添加@Override注解，在编译时，若该方法在父类中方法名相同、参数列表相同时（重写了该方法），则不会报错；若方法名不同（没有重写或重载）或者参数列表不同（没有重载）时，会报错；**

**即强制@Override注解该方法应该是对父类某方法的重写**

**5、向上转型，继承中，导出类可以自动转型为基类，即：**

new  导出类    ；可以直接作为基类对象使用    

Find    wind  =new  Wind()    （Find为wind的父类或超类）

当需要使用向上转型时（即想通过该类获取复用类类型对象，不仅仅复用该对象代码，还获取该对象整体形势）；则就需要使用继承

**6、向上转型，会忽视子类中所有的新方法（父类没有出现的方法名）；但父类的原方法和被重写的方法还是会被保留**

**7、当final修饰方法时，在之前版本（java5 之前）编译器会将所有的final方法的普通调用（将参数放入栈中，跳转至方法的代码执行，最后在清空栈中参数）转变为内嵌调用（复制保存方法的实际代码，当方法调用时，就按照该副本执行），这样就会消除调用方法的开销；但当方法代码量很多时，花费在复制代码的时间就会增加，其提升的效率也会变得不明显**

**在之后版本中，去掉了内嵌调用，开发者只有当需要方法不能被重写、重载时，才将方法设置成final**

**8、当final修饰对象类型的成员变量时，只是让该对象的引用不能改变，但还是能修改该对象引用指向的内存空间中的值**

**即该变量不能重写赋值，但可以进行对象操作来改变值，如数组、集合添加数据**

**9、类中所有的private方法都隐式指定final，当继承时，重写该private方法，并不会报错，原因是：**

private方法并不是父类接口的一部分（即private修饰方法并不会被继承），而是父类所隐藏的一些代码，因此继承后所写的该方法，只是名字相同而已，并没有覆盖；

**10、final修饰类，相当于使该类不能被继承，且所有方法都隐式被final修饰**

**11、final在修饰成员变量时，当该变量在为赋值时，会在对象创建时自动初始化，之后再被final修饰无法改变；但有时我们需要该变量保持不变，又希望每个该对象的该变量不一样时，我们可以通过在类的构造方法中来完成其变量的初始化**

我们会发现，此时会违反**成员变量要在构造方法之前初始化**的结论：因为当final修饰成员变量时，成员变量在构造方法执行前，已经被自动初始化，而final变量值就不能发生改变。实际上原因是：

**final修饰的成员变量不会进行自动初始化，必须手动初始化**

**12、导出类（子类）初始化顺序：基类（父类）静态成员变量------子类静态成员变量--------父类成员变量------父类构造函数-----子类成员变量-----子类构造函数**

**13、在使用继承时，会暴露父类的所有信息，但实际中只会用到父类的部分代码，这样大大减少了复用类的安全性，因此就出现了代理这种复用类方式**

代理：其实是对组合和继承的一种综合：**将复用类作为代理类的成员，而代理类有所有对应于代理类的方法（通过实现复用类同一个接口），实现各方法时，调用复用类对象成员的方法**

优点：相对于继承、组合，在使用代理时，即不会像组合暴露原类的所有方法；又不像继承，暴露原类的所有代码

**#在一般情况下，我们会选择组合（即我们常常用到的new一个对象，调用其方法），因为：**

1、继承耦合度太高，依赖于父类，并暴露父类代码，破坏了类的封装

2、java只支持单继承，无法满足我们对多个类的复用

3、代理在一般情况下是多此一举，会增加对代理类的管理

**14、代理一般是用来约束对该类的使用，并对该类方法代码的调用和扩展进行更格式化、规范化的处理**

## 多态

**1、多态，即一个类可以又多种类型的子类，但又可以向上转型为该类（父类）时，还可以保持自己的状态**

**2、类的多态，取决于类的方法不同；java的方法绑定有两种，前期绑定（静态绑定）、后期绑定（动态绑定）**

前期绑定：在编译期，就指定该方法为哪个类的；java当中的方法只有final，static，private和构造方法是前期绑定

后期绑定：在代码执行时，根据使用该方法的对象类型，来判断该方法名正确的方法体

即：**当子类向上转型为父类时，调用两者共有的方法时，使用的是子类重写后的方法，来体显类的多态性**

**3、在使用private、final修饰方法的类继承时，子类会将其屏蔽（在作为子类对象时，不会继承该方法）；看似对其重写的方法，只是写了一个新方法而已；子类向上转型为父类时，还是使用父类的原方法（被屏蔽的方法，会在向上转型时，显示出来）**

**4、被static修饰的方法，由于是前期绑定，也不具有多态性；静态方法只与类有关，不管该对象是通过实例哪个类创建的，只看对象的类型，来判断正确的方法体;静态方法是不能被final修饰的，因为final是为限制变量、方法的重写，或类的继承，而静态方法没有多态性，因此final就没有存在的意义**

**5、域（成员变量）也不具有多态性；如子类向上转型时，其成员变量不具有多态性，还是子类重新赋值的数据；但将成员变量数据放在方法中时，又由于方法的多态，导致其数据会根据对象类型而发生改变；**

因此为了防止这种情况，一般成员变量都通过private来修饰，防止成员变量在方法中体现多态性，来混淆代码；父类的成员变量通过super来获得，外部类通过get、set方法获得

**7、构造器（构造方法）也不具有多态性；因为构造方法隐式被static修饰，但只能通过new来调用**

**8、在java SE5中添加了协变返回类型：**

即之前子类重写父类的方法时，返回类型应该相同；但现在返回类型可以不同，但是子类的返回类型应该更具体（即返回类型可以为父类方法返回类型的子类）

9、在子类中通过super.xxx调用父类方法（包括构造函数），中间出现调用另外的方法时。该方法在子类重写，此时就会由于类的多态性，导致调用的父类方法中的使用的另外方法，不在是父类中的原方法；而是子类重写后的方法；**即在子类中，进行初始化或方法调用时，只有显式加上super才会直接调用父类方法，否则会都会调用子类重写方法**

## 接口

**1、抽象方法，当一个方法定义了其返回类型、参数列表、方法名，但没有方法体时，我们将其方法称之为抽象方法，并需要通过abstract关键词修饰；**

**#抽象方法在使用前，必须定义其方法体，因此该方法不能被private修饰，防止方法体不能被定义**

**2、当一个类（class）中存在抽象方法时，该类一定要被abstract修饰，并称为抽象类；**

抽象类可以没有抽象方法，但这样一般没有意义

抽象类在被继承时，必须定义其所有抽象方法（添加方法体）；否则子类也需要被abstract修饰，为抽象类

抽象类中可以有一般方法（包括构造方法）、成员变量

抽象类在被实例化时，编译器要求必须定义所有的抽象方法；当抽象类中没有抽象方法时，也必须添加{}来代表该抽象类中的抽象方法被定义**（该方式叫做匿名内部类实例化，但不是真正意义上的实例化，而是通过创建一个新Calss继承该类，然后new出来的实例化对象）**

**3、接口，相对于抽象类更为纯粹：**

**抽象类是为了让开发者在类中创建没有定义的方法（抽象方法），而接口是一个完全抽象的类，它只能够定义抽象方法**

接口是通过interface关键字来创建的，类似于class关键字的使用

接口的成员变量默认被static、final隐式修饰，因此必须手动初始化成员变量

接口中的成员变量、抽象方法都默认被public修饰（也只能被public修饰）；抽象方法默认被abstract修饰

接口中没有构造方法，但可以通过匿名内部类实例化方式来变相实现实例化，即通过其实现类来创建其接口类型对象

**4、接口和抽象类的使用区别：**

**抽象类的本质是复用类，通过继承来复用抽象类中的代码，重写其抽象方法来实现多态**

**接口是完全性的实现多态**

**5、接口能被另一个接口继承，来进行扩展；而一个类能实现多个接口，相对于继承的单继承性，有更好的扩展性能**

**6、当类同时使用继承和接口时，当存在方法名相同时；编译器会自动将继承中的方法看作是接口抽象方法的重写，此时若两方法的返回值或参数列表不同时，就会报错；因此我们要避免继承和接口出现这种情况**

**7、接口所设计的工厂模式：**

**通过某些条件参数，在工厂类中实例化接口的某个实现类，并以接口类型返回**

工厂模式的优点：通过工厂模式创建对象来代替new A，这样就可以用接口对象来代替new 出来的A对象；当A对象要替换成另一个实现类时，我们就不需要修该类中new A 的代码，只需要修改工厂类中要返回对象；这样就能通过接口实现类与类之间的解耦

**工厂模式有三种：简单工厂模式、工厂方法、抽象工厂**

简单工厂模式：就是上面这样，通过工厂中的方法来判断创建对应的实现类；实现类创建其接口类型对象（用在创建所需要的对象）

工厂方法：是每个实现类都有一个工厂类，这些工厂类有一个相同工厂接口，通过在外面来判断需要用哪个工厂生产对应的实现类；实现类创建其接口类型对象（用于创建子工厂类）

抽象工厂：相对于工厂方法，它可以扩展所有子工厂类中功能代码（添加另一个实体类）

即在总工厂接口中，定义返回两个实体类的两个不同接口的抽象方法，根据子工厂类的不同，来实例化这两个方法，并返回对应的实体类对象；

## 内部类

**1、内部类：可以将一个类定义在另一个类的内部，定义的类叫做内部类，另一个类叫做外围类**

若从**外围类的非静态方法外的方法（外围类静态方法、其他类方法）**中，创建内部类的对象时，该对象类型应写为**外围类名.内部类名**

**外围类的非静态方法可以直接通过new内部对象来创建**

**2、通过内部类，可以操作外围类中被private修饰的成员变量和方法**

在创建内部类对象时，编译器会自动隐性获取该内部类的外围类对象的引用，从而操作其对象的成员变量；此时在**内部类中使用外围类成员变量或方法，是直接通过变量名或方法名来访问**

但实际中会出现外围类和内部类成员变量相同情况，因此需要使用this关键词来显式区分对象的引用；通过**外围类名.this**   来作为外围类对象的引用

**3、内部类对象的创建必须通过外围类对象来完成（排除静态内部类），一般有两种方式：**

通过外围类方法来返回内部类对象

通过外围类对象和new关键词来创建内部类对象，即  **外围类对象名.new    Inner（）**

**内部类（排除匿名内部类）和普遍类一样可以重载构造方法**

**4、内部类修改权限修饰词时，和类中的成员一样，访问权限也会改变**；如：

内部类被private修饰时（**私有内部类**），在外围类除外的类方法中   **外围类对象名.new  Inner()**  就无法被执行

**通过外围类方法创建的私有内部类对象只能向上转型为其实现的接口（或继承的父类）类型，这样就使得该内部类对象只能使用公共接口中的方法，而其他扩展方法只能通过外围类执行调用，实现完全隐藏类的实现细节。**

**5、在一个方法中或者任意作用域中定义的类，被称之为局部内部类；**

**局部内部类只能创建使用在该作用域中，因此不能使用任何权限修饰符**

**局部内部类只能访问作用域中被final修饰的变量**

**6、在方法中创建一个类型对象时，并对其进行类的重定义（实现对应类型的接口、继承对应类型的类），此时该类没有名字，但还是在一个类中定义了另一个类，因此叫做匿名内部类**

匿名内部类的定义语法是对   **定义一个实现类（继承子类），并创建其对象**   过程代码的简化；让实现类的定义编写在创建对象时的代码中，方便代码维护

**匿名内部类只使用在方法创建对象语句中，因此不能使用任何权限修饰符**

**匿名内部类中没有构造方法，而是使用继承的基类构造器（接口式的匿名内部类是继承的object类的默认构造方法）**

**匿名内部类和局部内部类一样，所在作用域中的使用的变量必须是被final修饰的，原因是：**

作用域中的局部变量生命周期和内部类中的变量生命周期是不一样的。内部类在使用了作用域的局部变量，作用域执行结束时，其局部变量会销毁，但其内部类对象却不一定销毁，这样就会出现内部类对象中引用了一个已经不存在的内存空间

为了解决这个问题，就必须保证变量值的内存空间不被销毁，在内部类使用时copy一份变量引用，使两个引用同时指向一个内存空间，当作用域的引用消失时，内部类的引用还是能访问到对应内存空间；而能让变量一直指向同一个内存空间的前提就是，该变量是被final修饰的，其引用就不会发生改变，

**匿名内部类有个特例，当变量在其父类的构造方法中使用时，不需要被final修饰**

**7、当内部类被static修饰时，被称为静态内部类（也叫嵌套内部类）**

**相对于非静态内部类来说，编译器不会自动隐式保存外围类对象的引用，当然也不能手动使用this来获取外围类对象的引用**

**它的创建不需要外围类对象，也不能对外围类的非静态成员方法进行访问；但可以访问外围类的静态成员、方法**

**8、内部类可以存在多层嵌套，并跨层访问外围类成员变量、方法；接口中也可以编写内部类，来实现该接口**

**9、内部类被分为四种常用类型，及其常用场景：**

成员内部类  ：用于实现多重继承的解决，通过多个内部类来继承一个类（接口），然后再该内部类中调用继承、重写的方法操作外围类的成员变量，从而实现多重继承的代码复用；且当多个内部类继承同一个接口时，可以不同方式实现其方法，方便清晰的展现类的多态性

局部内部类 ：用于减少类文件的产生，当该作用域所用代码需要封装成类，但又只需要该作用域使用时，可以使用局部内部类的形式，让代码更容易维护；且局部类对于外围类是不可见的，这样极大增加了内部类的安全性

匿名内部类 ：其作用和局部内部类相似，是一种特殊的局部内部类，但其只能再定义时实例化创建一个该对象；而局部内部类可以通过new 来创建多个对象；一般使用在绑定事件、创建多线程、回调函数代码中，只需要创建一个对象，简化代码。

静态内部类 ：不需要外围类进行创建，只给外围类提供服务，但逻辑上可以单独存在；一般只用于外围类的单元测试：正常每个类测试需要编写main方法，这样打包前还有一个个删除，当每个类使用静态内部类存放主程序时，由于内部类的特点，会自动将这些存放主程序的类编译成要成一个个单独的class文件，我们就可以通过运行这些文件，一个个测试，最后打包时将这些文件删除。

**10、内部类标识符：在含有内部类的类在编译时，会生成一个外围类class，和多个内部类class；内部类class文件名为：   外围类名$.内部类名**

**当内部类为匿名内部类时，则为   外围类名$X               X为正整数**

**当内部类嵌套在另一个内部类时，     外围类名$.内部类名.子内部类名**

11、内部类的继承有两种类型：

内部类被另一个类继承：此时内部类和外围类是独立的两个类，但是必须在另一个类中的构造方法中添加被继承内部类的外围类的构造方法

public class test1 extends test.Kiss{

public test1(test test) {

//super（）代表test对象的父类构造方法（可以是无参或有参）

​	test.super();
}

即test1类的创建必须通过test类的实例来实现

内部类被另一个类的内部类继承：一样四个类（俩个外部类、俩个内部类），除了继承的类之间有联系，实际上都是独立的类；此时可以通过外围类继承，从而使内部类获得所继承内部类的外围类对象的引用，此次该内部类的创建就不需要通过继承内部类对应外围类对象了；

**内部类不会像外围类的方法、成员变量一样，随着外围类的继承而再写一个同名内部类时，而被覆盖；实际上这两个内部类是独立的，会根据外围类对象的类型（子类还是父类），从而操作它们对应外围类的成员变量和方法**

**12、内部类的闭包特性：通过一个可以调用的对象（内部类），来保存    它被创建时所在的作用域  中的一些信息（如获取外围类成员变量）**

**通过内部类的闭包功能，可以实现回调；在内部类中的方法，通过外围类对象参数来回调执行外围类中的成员、方法**

## 持有对象

1、java持有对象即为容器类（集合）；用于增强对对象的使用，解决数组只能创建保存固定数量对象的致命缺点

2、容器主要分为两种类型，对应接口**Collection与Map**，它们定义了两种不同的对象存储方式

**Collection**用以保存单一的元素，**Map**保存关联键值对。通过**泛型**来指定容器存放的数据类型。

**Iterator** 设计的目的是在未知容器具体的类型的情况下，用来遍历容器元素；**collection接口实现类才能创建iterator对象**

3、容器类在使用时，常只使用默认接口（List、Map、Set、Queue、Iterator）中限有的方法，并使用的是接口类型对象，因此一般容器类创建时应向上转型        List<E> list = new ArrayList<E>();    

但要使用容器实现类的额外方法时，就不能使用向上转型

4、**给Collection对象（包括其子类对象）添加数据**：

​	1、Collection对象在创建时，可以通过已有的实现类对象进行初始化（这种方式运行效率低）

​	2、Collection对象本身执行addAll方法添加  （效率一般）

​	3、通过Collections工具类，为Collection对象添加（效率最高）

​		collections.addAll(collection,数据对象)

5、Arrays工具类提供asList（）方法，将数组转化为list对象；但是会有一下缺点：

​	1、数组类型不能是基础数据类型，原因：list泛型中没有基础数据类型

​	2、转化的list对象使用remove、add方法时，会出现运行时报错，原因：转化的list对象是Arrays中的内部类实例，并不是list接口实例，因此不提供remove、add方法

6、Collection子类接口有：List、Set、Queue；这些接口根据作用不同，相对于Collection增加了额外方法

​	1、List分为ArrayList和LinkedList，是有序的Collection接口；

​		保存元素顺序：两者都是按照插入顺序保存

​		使用性能：ArrayList在随机访问时的效率高，中间插入、删除元素效率低；

​                           LinkedList为链表结构，适合顺序访问，随机访问效率低，但进行插入、删除效率高

​	2、Set分为HashSet、TreeSet、LinkedHashSet，为不重复的Collection接口；

​		 保存元素顺序：HashSet保存是通过hash值决定保存位置，因此顺序随机；TreeSet根据数据类型（integer类型默认按照大小排序。。。）来进行排序保存；LinkedHashSet在HashSet的基础上，使用链表结构维护元素的次序，进而看起来（打印、遍历）使元素按插入顺序保存

​		使用性能：HashSet、LinkedHashSet都使用了**散列函数**来计算hashCode，来决定保存位置，因此访问速度很快。TreeSet使用红黑树，对数据进行排序保存。

​		**在插入时由于linkedHashSet需要维护链表结构，因此效率比HashSet低一些**

​		**treeSet不能存入null，因为值需要调用Comparator接口实现方法进行排序，若出现值null，则会发生NPE**

​	3、Queue有两个实现类：LinkedList、PriorityQueue，队列容器，可以让容器中的对象从一端放入、出另一端取出，实现先进先出让对象从一个区域传输到另一个区域

​		LinkedList实现了Queue接口，但有List接口的其他方法；Linkedlist使用的使Queue默认的队列规则：先进先出

​		PriorityQueue则是一个更高级的队列，可以自己定义队列规则，根据元素的优先级来输出下一个元素；队列规则通过自身提供的Comparator对象来设置

7、Map接口有三个实现类：HashMap、LinkedHashMap、TreeMap；它们的特征和Set的三个实现类一致

**因为Set接口的实现类操作，底层使用的是Map实现类来完成**

**HashMap、LinkedHashMap允许key、value为null；TreeMap由于需要对key进行排序，因此key不能为null**

8、容器对象常用方法：

​	Collection接口常用方法：

add(E e) ，添加元素
clear() ，暴力清除集合中所有元素
contains(Object o)， 返回值类型：boolean。判断集合是否包含某个元素
isEmpty() ，返回值类型：boolean。如果此集合不包含元素，则返回true。即用来判断容器是否被赋值过
iterator() 迭代器。返回值类型：Iterator
size() 返回值类型：int。返回集合中的元素数

​	List接口额外常用方法：

get（int index）：获取指定索引位置元素
set（int index，Object obj）：修改、删除、添加指定索引位置元素

Set接口和Collection接口方法完全一样

​	Map接口常用方法（Map 集合没有继承 Collection 接口，其提供的是 key 到 value 的映射）：

| 方法                          | 功能描述                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| put（K key，V value）         | 向集合添加指定key-value数据                                  |
| size（）                      | 获取集合中的元素个数                                         |
| clear()                       | 清除集合中所有的元素                                         |
| remover（Object key）         | 删除指定元素                                                 |
| isEmpty（）                   | 是否为空集合，是则返回true                                   |
| containsKey（Object key）     | 查询集合是否包含该key，有则返回true                          |
| containsValue（Object value） | 查询集合是否包含该value，有则返回false                       |
| get（Object key）             | 通过key对象返回对应value值，没有则返回null                   |
| keySet（）                    | 获取所有key对象的Set集合                                     |
| values（）                    | 获取所有value对象的Collection集合                            |
| entrySet（）                  | 获取所有key-value数据视图的Set集合（提供iterator迭代器进行遍历） |

9、Stack是一个元素先进后出的容器，继承了LIST接口，常用额外方法有：

s.push("a");  入栈
s.peek（）; 查看栈顶
s.pop(); 出栈

除了可以按照LIST方法存放数据外，还可以使用这些额外方法来实现栈的数据保存方式（先进后出）。它继承了Vector，底层为动态数组。

10、迭代器iterator，通过next（）方法获取下一个元素，hasNext判断是否还有数据，从而不用关系容器大小；ListIterator是iterator的子类，对其方法重写，专门针对于List接口类型使用；提供了双向移动，能获取当前元素的前后两个元素

**List拥有一个增强版迭代器（listIterator），它还可以调用set（）、add（）方法，在当前遍历位置替换、插入元素**

**在普通迭代器执行时，通过iterator 的remove方法,将当前对象从容器中移除，并且可以对当前遍历的对象进行操作，从而影响容器中对象的值**

11、foreach循环，内部是使用了iterator进行遍历；当在foreach循环中使用add、remove方法时，会出现错误
原因：foreach循环时，会创建对象的iterator对象，此时会保存当前对象的modCount值（容器修改次数），每次进行next方法时，都会监测对象的modCount和iterator的expectedModCount是否一样，不一样就会报错。
而foreach循环中进行add、remove时就会改变对象的modCount值，此时就会报错ConcurrentModificationException

同样的list接口中的sublist也有这个问题，sublist生成的容器是原容器的视图，当我们操作原容器元素时，子容器也会发生改变，但子容器中的modCount值不会变；当操作子容器时，原容器也会改变，并且原容器的modCount值也会变；由于子容器的属性要确保modCount值和原容器一致，因此当原容器元素个数改变时，两者的modCount值就会不同，此时再进行视图对象的任何操作，都会报错ConcurrentModificationException

12、Map的遍历方式：

Map在遍历时，也可以对遍历的对象进行操作，从而改变map中的值

java8下，推荐使用foreach方法执行

  hashMap.forEach((k,v)-> {System.out.println(k+""+v);});
	}

**foreach（）方法内部使用了lambda表达式实现，本质是一个匿名函数（匿名内部类），因此无法使用函数外面（外部类）的非final局部变量**

java8以下版本，推荐entrySet foreach遍历

    Set<Entry<K, V>> entrySet = hashMap.entrySet();   
    for (Entry<K, V> entry : entrySet) {
    	System.out.println(entry.getkey+""+entry.getValue);
    }
13、foreach和itreator的选择：

foreach需要知道容器具体类型，且不能使用remove、add方法，但是代码整洁、易懂

itreator不用知道容器具体类型，且速度快，且可以遍历删除remove当前元素

14、ArrayList、Vector底层是动态数组，根据默认的长度进行一半或者一倍的增加：

因此对于这两个容器，推荐指定默认长度，防止过多的长度增加工作，损耗效率

15、Vector、Stack和hashtable 它们都是线程安全的（添加同步锁），但它们已经过时

过时原因：

1、由于这些集合的创建是在JAVA1.0/1.1中，当时没有系统的集合框架，在方法使用和效率上都没有优化
2、在java5后有了单独处理多线程的集合（在concurrent工具包中），在安全同步上优化更好

![](C:\Users\OneMTime\Desktop\Typora图片\JDK8集合API架构图.png)

16、对于数组和持有对象（LIst、map、set）添加元素时：

1、对于基础数据变量（int、long。。。）添加的是数据本身（变量指向栈中的数据）

2、对于对象，添加的是数据的内存地址，即数据的引用（变量指向栈中的对应对象的引用）；这样会导致在集合在重复通过该变量添加数据（每添加一次数据，就修改该变量对象的值）时，会导致集合中保存的都是同一内存空间，全部数据相同

​	 对于final修饰的对象（基本数据类型包装类String、Integer。。。）在修改时，其实是创建了一个新对象

​	 对于对象变量直接赋值时，其实是修改了变量保存的引用

## 通过异常处理错误

1、创建自定义异常：

​	继承Exception或RunTimeException，定义构造方法，super("自定义错误");来定义该异常信息（也可以使用有参构造器来定义）

2、手动抛出异常：

throw new MyException() ,此时方法需要其进行捕获或者向上抛出

#springboot框架中产生异常时，是通过请求异常处理器来进行处理，因此带代码中不用对异常进行处理

3、try --finally由于finally一定会被执行，因此可以用来在多个if--return中进行资源关闭

在创建需要清理的对象后，应马上进入try--finally语句块；而创建对象的代码应用上一级try--catch处理

原因：创建对象时，可能会出现对象构造器执行异常，导致finally中的关闭语句无法执行

4、异常catch语句匹配：
当catch基类异常和子类异常时，应将子类异常放在前面（且两个异常都会匹配）
若基类放在前面时，后面的子类将不会匹配

5、异常漏洞：

当try--catch嵌入try--finally时，当try--finally捕获异常，在finally中又抛出异常后，上一级try--catch只会捕获到finally抛出的异常；这种情况就是异常丢失。

6、finally使用常见问题：

​	1、当finally中使用return语句时，由于finally一定会执行，所以当try中有return时，finally中使用的return会将其代替。

​	2、finally不是一定会执行，当代码在try之前产生异常时，或者try中代码执行突然终止时，如执行System.exit(0);

7、throw和throws的区别：

throw是用于代码中，手动抛出（创建）一个异常实例，从而进行异常处理
thorws是用在方法上，用于将异常抛出，给该方法的调用代码进行异常处理

8、一般情况下，在finally中使用return语句时，会出现警告；因此return语句应写在finally语句后面

## 字符串

1、String对象是不可变的，每次对String的操作，实际上是重新创建一个String对象；当String操作没有发生内容变化时，String方法并不会返回一个新String类，而是返回原来的引用。

​	String类没有append方法

2、java中不允许程序员重载任何操作符，但用于String对象的“+”和“+=”是被java默认重载的，执行了StringBuilder的append（）方法，然后转换为String对象

3、由于String类操作时会产生很多无用的中间String对象，大大增加了内存损坏和垃圾回收

java5后，提供了StringBuilder类，用于更效率的对字符串进行操作；
java5之前，使用StringBuffuer类，其线程是安全的，因此效率比StringBuilder低一点

4、当String对象拼接其他类型对象时，会自动调用其对象的toString方法；当重写了该对象的toString方法，返回对象的内存地址时，不能直接调用对象的toString（会出现递归），应该调用Object的tostring方法，即super.toString()

5、java5后，提供了格式化输出

system.out.printf(）  C语言风格    system.out.format()   java风格

system.out.format(“%d       +           %f”，x，y    )    前面为字符串  ，后面为需要格式化的参数，根据顺序与字符串中的格式修饰符对应。
通过%+数字+s），可以设置字符占位数，即字符串最后一个字符距离上一个字符串占位数；来控制每个字符串之间的空格数（当距离无法满足字符串本身长度时，则将两字符串相连）

6、文本格式化依赖于Formateer类，通过该类的format（）方法来进行String类型和基础数据类型的格式化
String类也提供format（）方法，但只能格式化String类型参数

7、String类的使用正则表达式的方法：

​        split（），将字符串在满足正则表达式的地方分隔
​	matches（），将字符串与正则表达式匹配，满足则返回true

8、java提供java.util.regex.Pattern类来创建正则表达式处理对象，并提供更多的匹配后的处理方法

```java
//    通过Pattern类来编译正则表达式，生成模板对象
Pattern pattern = Pattern.compile("\\W+");
//    使用模板的matcher方法来匹配指定字符串，生成匹配器对象
Matcher matcher = pattern.matcher("helloword");
```
9、Matcher对象提供对正则表达式的使用：
	find（）和group（）方法用来返回匹配成功的字符串

​	end()、start()方法返回当前匹配所在字符串中的

```java
//    find方法判断是否有匹配的字符串(当匹配后，会排除掉匹配的字符串，继续往下匹配)
    while(matcher.find()){
//    group用于输出当前匹配的字符串	
    	System.out.println(matcher.group()+"起始下标"+matcher.start());
    }
```

​	reset（）方法提供重新指定要匹配的字符串

10、Scanner 扫描输入类（JAVA5提供）：其构造器可以接受任何类型的输入对象，然后通过方法以特定的方式输出

​	1、new Scanner()可以接受的输入有：

file、InputStream（如最常用的System.in）、String、path、readabel

​	通过Scanner方法，可以将其以某种方式扫描处理

​	2、Scanner类还可以结合正则表达式使用

​    3、在java5中没有Scanner、正则表达式；使用StringTokenizer对字符串进行处理

11、字符串三种常用对象使用：

1. String：它被用于裁剪，拼接。(当然如果拼接过多的话还是建议用StringBuffer,或者StringBuild)搜索字符串，比较字符串，截取字符串，转换大小写等。在项目中不经常发生变化的业务场景中，优先使用String
2. StringBuffer：用于拼接，替换，删除。在项目多线程环境下运行，如:XML解析，HTTP参数解析与封装等。
3. StringBuilder:它同StringBuffer使用方式一样，不过在项目中使用的地方建议是单线程的环境下，如：SQL拼接，JSON封装等。

## 类型信息

1、为什么需要RTTI（Run-Time Type Identification 运行时类型识别）

​	当展现java多态时，子类可以通过向上转型创建为父类对象，但其实引用了子类对象的实例；此时调用的应该是子类对象方法（父类中有的），因此就需要RTTI来识别对象变量的类型

2、RTTI提供了什么信息

​	RTTI可以识别出该变量对象实际对应类的class对象

​	1、每个类都有一个class对象，在jvm的类加载器子系统中生成
​	2、每当类被编译后，就会生成类加载是创建Class对象的代码，并保存在编译的class文件中
​	3、只有当类被使用时才会被加载，这个称之为动态加载；加载时就生成class对象
​	4、类的使用即为对静态成员的引用：构造器（new，说明构造器也是静态方法）、静态变量、静态方法
​	5、只有类的class对象被创建后，才能使用new来创建该类对象

3、Class对象的获取方法

​	1、Class.forName(“包名.类名”)       通过类的全限定类名来获取该类Class对象

​	2、Student   s=new   Student（）；  s.getClass()    根据类的一个实例对象获取Class对象

​	3、Student.class             根据类名+.class关键字  来获取该类的class对象

这三种方式的区别：

​	      Student.class方式会让该类被加载，但不会进行初始化，尽可能实现延迟加载；满足需要使用时，才处理数据；
​          Class.forName的方法在类被加载后，会进一步将其类的静态成员初始化（不包括构造器）
​	      通过new的实例来获取Class对象时，会按照类的正常使用进行：加载----链接----初始化（包括静态和非静态）

4、对Class对象添加泛型

原因：让Class对象可以进行编译期类型检查，不需要等到运行时才检测类型，防止我们无意间进行错误操作，编译器可以有错误提示

添加泛型方式：

​	1、Class<?> ，使用?通配符，表示可以是各种类的Class对象；本质上和不使用泛型一样，但这样不会产生编译期警告信息（未指定该Class的type），并且可以提示开发者该变量用于引用非具体类，可以随意选择其他类Class对象的引用

​	2、Class<? extend  Father>  使用？通配符和extend关键字，限制类型范围。指类型为Father类及其子类

​	3、Class<? super  Father>  使用？通配符和super关键字，限制类型范围。指类型为Father类及其父类

5、Class对象的使用

​	1、getName（）返回String形式的该类的名称
​	2、newInstance（）根据某个Class对象产生其对应类的实例，它调用的是此类的默认构造方法(没有默认无参构造器会报错)
​	3、getClassLoader（）返回该Class对象对应的类的类加载器
​	4、getSuperClass（）返回某子类所对应的直接父类所对应的Class对象
​	5、getInterfaces（） 返回该Class对象对应的类的实现接口
​	6、isArray（）判定此Class对象所对应的是否是一个数组对象

6、Class对象和instanceof关键字判断对象类型

instanceof会考虑类的继承关系，即 x instanceof   A  ，true时，X是A的相同类或者子类

通过类的Class对象比较，判断他们对应的类是否为同一个类时，可以使用==或者equals（），为true时，代表两个对象为同一个类

7、对象类型测方法：instanceof、isInstance、isAssignableFrom

instanceof为关键字，表达式为     实例A   instanceof   实例B；支持向上转型

isInstance、isAssignableFrom为Class对象的方法；前者参数为类的实例对象，后者为类的Class对象；且不支持向上转型，即调用方法的对象不能匹配父类

8、注册工厂

​	注册工厂创建过程：

​	1、创建一个生成器类，将所有的对象工厂类实例放到一个MAP集合中，之后根据需要的对象名作为KEY来获取对应工厂类实例，然后执行该工厂类的create（）方法
​	2、在需要创建的类中，提供一个可以返回自己实例的静态内部类，实现Factory接口，提供create（）方法返回外部类实例

​	为什么使用注册工厂：有时需要隐藏类对象的创建过程，因此将其写在自身的类中，并提供一个固定的模式来获取对象（可以获得不同类型的对象）

9、反射机制

​	java反射定义：

​	在运行状态中，对于任何类都可以获取其方法和属性；将这种动态获取对象信息和调用对象方法的功能称之为反射机制

​	反射的使用方法：

​	1、获取类的Class对象
​	2、通过class对象的getConstructors()方法获得所有构造方法
​	3、通过class对象的getDeclaredFields()方法获得所有属性
​	4、通过class对象的getDeclaredMethods()方法获得所有普通方法

这些对象都需要setAcessible（true）来关闭安全性检测，获取private修饰的类成员

​	使用反射的原因：
当不使用反射是，所有对象都需要new来创建，当需要换另一个类是是实例化时，就需要修改源代码重新编译；当使用反射后，通过类名来在运行时动态创建，就可以将类名放在配置文件中修改，更加灵活快速

10、动态代理

动态代理对象创建通过Proxy（java.lang.reflect.Proxy）的newProxyInstance方法创建

参数为：被代理类的类加载器对象、代理类的所有实现接口，invocationHandler实现类对象

```java
		JavaDev javaDev = new JavaDev();
	Developer dogProxy = (Developer)Proxy.newProxyInstance(javaDev.getClass().getClassLoader(), 
			javaDev.getClass().getInterfaces(), 
		//实现invocationHandler接口的匿名内部类（重写一个方法，用于对被代理类方法的处理，通过lambda表达式写）//				代理类对象，需要代理执行的方法信息对象，方法参数数组
			(proxy,Method,agrs)->{
				System.out.println("进行动态代理");
 				Object invoke = Method.invoke(javaDev, agrs);
                System.out.println("进行动态代理");
                //invoke为javaDev实现类的当前方法的返回值；当return null时，代理类执行返回值就为null
				return 	invoke;				
				});
//		通过动态代理对象执行方法，调用其invocationHandler接口的实现类方法，并返回
		dogProxy.code();
```
​	动态代理是在不确定被代理类的情况下，通过反射机制来动态获取被代理类信息，从而创建动态代理对象，并编写对应其方法执行处理

​	动态代理对象类型只能指定为接口类型，然后可以在运行时动态创建对应实现类的代理对象

​	动态代理和静态代理的区别：

　静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了
　动态代理：在程序运行时，运用反射机制动态创建而成

11.空对象模式

​	空对象模式使用环境，一般与工厂模式联用：

当创建一个对象时，由于参数等不合法，导致对象创建不成功变为null，之后调用其方法或属性时就会报空指针异常；因此为解决这种问题，就可以使用空对象模式（当然也可以判断对象是否为null，但这样会增加代码量，每当使用创建该对象时，都需要判断，不满足代码的复用）

​        空对象即继承对应类接口，并实现对应方法，返回值用来提示该对象为null；在工厂类的构造方法中，当参数不合理时，返回空对象实例；合理则返回对应对象实例

## 泛型

1、简单泛型（泛型类）的作用:

​	1、指定容器类存放的对象类型
​	2、动态指定类的实例对象中的成员变量类型（用来代替通过object类型的成员变量实现多个实例使用不同类型成员变量）  一般单个成员变量使用T来代替

````java
public class Dog<T>{
    private T name;
    public Dog (T t){name=t}
}

Dog dog=new Dog<String> ("hhh")
Sting name=dog.getname()    // String 类型  hhh    
````

2、泛型接口，用于动态指定该接口中方法的返回类型；当接口被实现时，指定泛型类型，从而确定实现方法的返回值类型，（一般用于工厂模式创建对象，即生产器）

```
public interface Generator<T> {
    public T next();
}

public class FruitGenerator implements Generator<String> {

    private String[] fruits = new String[]{"Apple", "Banana", "Pear"};

    @Override
    public String next() {
        Random rand = new Random();
        return fruits[rand.nextInt(3)];
    }
}
```

3、泛型方法

```java
 *     1）public 与 返回值中间<T>非常重要，可以理解为声明此方法为泛型方法。
 *     2）只有声明了<T>的方法才是泛型方法，泛型类中的使用了泛型的成员方法并不是泛型方法。
 *     3）<T>表明该方法将使用泛型类型T，此时才可以在方法中使用泛型类型T。
 *     4）与泛型类的定义一样，此处T可以随便写为任意标识，常见的如T、E、K、V等形式的参数常用于表示泛型。
 */
public <T> T genericMethod(Class<T> tClass)throws InstantiationException ,
  IllegalAccessException{
        T instance = tClass.newInstance();
        return insance;
}
```

泛型只能指定引用数据类型，不能使用基本数据类型（使用包装类代替）

## 数组

1、数组与其他种类的容器相比，是随机访问、储存效率最快的，但付出的代价是数据对象的大小是固定的；

ArrayList是作为数组无法改变大小的代替品，实现空间动态分配，但效率低很多

**在泛型没有提出前，数组还有一个优势：可以定义储存对象的类型，进行编译期检查**

2、Arrays是java原生提供操作数组的工具类，常用方法有：

asList（a） 将数组转化为list，不推荐使用，有很多风险
Arrays.equals(a1,a2)   比较两个数组是否相同
Arrays.sort(a,string)    使数组元素按照某种方式排序
Arrays.binarySearch(a,value) 通过二分法来寻找指定值在数组中的index，但是要求数组元素是已经排序的（由于内部使用的二分法，因此一定是排好序的）
Arrays.fill(a,value)  将一个值插入到数组所有位置,当然也可以设定起止点来插入一段相同值（一般用于快速创建测试数据）

3、数组目前来说是应该被容器类替代，但多维数组的作用是无法代替的，它能简单实现线性代数的运算（矩阵）

## 容器深入研究

1、填充容器方法：

​	1、通过Collections.nCopies来指定容器实例化时，元素个数和所有元素的值（单一相同值）
ArrayList<String> arrayList = new ArrayList<String>(Collections.nCopies(5, new String("nihao")) );

​	2、通过Collections.fill对已经实例化赋值过的容器，替换所有位置的值
Collections.fill(arrayList, new String("CNM"));

2、java容器类采用快速报错机制，当容器被多进程同时修改使用时，会直接抛出ConcurrentModificationException异常

java5后提供更优化的同步容器：**ConcurrentHashMap、CopyOnWriteArrayList、CopyOnWriteArraySet、ConcurrentSkipListMap**

3、离散容器的使用，通过对容器中的数据进行哈希运算，得到哈希表，每一个对象都在哈希表中有唯一值，这样就能通过哈希值（离散值）快速找到对应的容器值

当我们需要比较两个对象是否相等（即两个对象实例的数据一致）时，就需要重写对象的equals方法，否则就是比较两个对象的引用地址是否相等。

​		equals重写思路：

​		对于我们自定义对象来说：1、先判断两对象是否相等，即判断它们引用地址是否相对，为同一个对象
​													  2、判断参数对象是否为null，满足equals方法的非空性
​													  3、判断前一个对象为后一个对象的同类或者子类，若成立就继续判断各个属性值是否相等，相等就对象相等

4、由于hash值在使用时，应满足对象相等则hash值相等，因此在重写equals方法后，hashCode方法也要重写

​		hashCode重写思路：1、由于决定对象相等的时属性值，因此将所有属性值作为参数
​											 2、公式为： int   result  =31*result  +filedHashCode()		

​		以String的hashCode为例：将字符串转为字符数组，并所有字符转为ASCII码，进行运行，得到唯一hash值

为什么要使用31作为翻倍参数：

​		1、31是一个不大不小的质数，一百以上的质数哈希值会超出int范围，太小如2的质数，会出现hash值冲突（不唯一），而使用常数 31, 33, 37, 39 和 41 作为乘子，每个常数算出的哈希值冲突数都小于7个

​		2、31在JVM进行位运算时更高效：

​          31*i=（i<<5）-i   ,该式可以简化乘积运算

5、需要重写equals、hashCode的常见场景：

由于set存储的是不重复对象，因此会调用存储对象的hashCode和equals进行判断，因此set存储的对象需要重写equals、hashCode方法；

map中的键使用的对象也一定需要重写equals、hashCode；String类就重写了equals、hashCode，因此一般使用String作为map键

6、通过使用set的不可重复性对一个集合进行去重，免去对集合遍历、对比、去重的操作过程

7、集合转数组时，不要直接使用无参的toArray（）方法，这样只会得到一个Object [],当强行转化其他类型时，会出现异常，无法转化

 因此应使用方式：		List<String> arrayList =new ArrayList<String>();
		arrayList.add("123");
		arrayList.add("1234");
		arrayList.add("12345");
		String[] array=new String[arrayList.size()];
	    array= arrayList.toArray(array);

## JAVA I/O系统

1、java通过流的抽象概念来表达、简化I/O类处理数据的过程；InputStream、OutputStream是最基本的I/O流

​		InputStream对象根据读取数据源的不同，有不同子类来表达输入流

常用：ByteArrayInputStream          读取内存中的字节数组        
		    StringBufferInputStream     读取String对象（被java1.1的Stringreader将其取代，已弃用）
			FileInputStream                     读取文件																						

不常见：pipedInputStream     管道输入流
               SequenceInputStream  将多个inputSteam合并成一个单一的inputSteam

​		OutputSteam对象根据写入的数据容器不同，有不同子类来表达输出流

常用：ByteArrayOutputSteram         写入到内存中，以字节数组保存
			FileOutputStream                 写入到文件中

不常用：PipedOutputStream    管道输出流

2、对于输入、输出流可以对其进行包装，优化I/O流传输；它们统一继承了FilterInputStream（FilterOutputStream），对于inputStream（OutputStream）进行进一步封装处理。

常用：
DataInputStream /DataOutputStream       基础数据流，使流中的数据以基本数据类型进行读写，不需要将数据先转化为字节类型保存，方便数据的读写，如通过DOS.readInt（） 
BufferedInputStream/BufferedOutputStream      缓冲流，使用缓冲区，将数据先全部读取/写入到缓冲区，然后再读取到内存（写入到文件）
PrintStream    打印输出流，用于对数据进行格式化输出（只能支持8位字节流（一个字符一个字节），对于16位的Unicode字节流（一个字符两个字节）可能出现乱码）

不常用：
LineNumberInputStream    对于读取的数据，没读取一行时，添加一个行号；从而可以快速获取对应行号的数据，由于在读取过程中，我们可以方便的添加对行数的标记，因此该包装流不常用
PushbackInputStream  用于弹出一个字节的缓冲区，我们不可能使用到，用于java编译器读到最后一个字符后回退

3、java1.1后，添加了Reader字符输入流、Writer字符输出流；之前inputStream、outputStream都是以字节数组进行数据传输，而Reader、Writer是以字符串来进行传输

新增的包装类（FilterInputStream（FilterOutputStream）的子类）InputStreamReader/OutputStreamWriter可以将字节流转化位字符流

新增包装类FileReader/FileWriter，来直接获取文件对象的字符输入流/输出流

字符流新增的原因：传统I/O流只能支持8位字节流，不能很好的处理16位的Unicode字节（全球统一字符编码）。因此需要使用字符流来支持Unicode；并且字符流比字节流操作更快，效率更高。

4、java1.1后，对于printStream添加了PrintWriter（字符打印流），对输出流进行装饰，用于优化对字符数据的打印；

5、当使用readLine()方法时，一定选择BufferedReader，不能使用DataInputStream，因为该输入流的readLine（）方法只能有效的将UTF-8的编码字节转化为字符串，其他编码的文件字节转化时会出现乱码；而BufferedInputStream由于为字节流，因此没有readline（）方法。

需要使用read（）方法时，推荐使用DataInputStream

6、基本输入流只能使用read方法，一个一个字节读取；只有BufferedReader或DataInputStream包装过的输入流才能使用readline（）方法

7、I/O流典型使用方式：

​		1、当想要读取一个文件，并已字符串读取时

````java
			BufferedReader bufferedReader = new BufferedReader(new FileReader(new File("d:\\dc.txt")));
			String s;
			StringBuilder sb=new StringBuilder();
			while((s=bufferedReader.readLine())!=null) {
				sb.append(s+"\n");
			}
			bufferedReader.close();
			String result=sb.toString();
````

​		2、获取到的字符串使用StringReader读取，并打印

````java
StringReader in = new StringReader("nihao");
		int c;
		while((c=in.read())!=-1) {
			System.out.println((char)c);
````

​		3、通过输入输出流来完成文件的读写

````java
		//在字节流创建时，会自动读取全部数据的总字节数
		BufferedInputStream bi = new  BufferedInputStream(new FileInputStream(new File("d:\date.txt")));
		BufferedOutputStream bo = new BufferedOutputStream(new FileOutputStream(new File("c:\test.txt")));
		
		byte[] b= new byte[1024];
		int i =0;
//当执行read()方法时，文件会通过系统i/o读取部分数据存放在缓冲区中；
//当缓冲区数据读完后，再次使用系统I/O来读取文件，装满缓冲区（文件读完时停止），再进行缓冲区数据读取
		while((i=bi.read(b)) != -1) {
//此时是将数据写入到缓冲区（当缓冲区数据装满时，自动触发磁盘i/o写入，将数据写入到文件）
			bo.write(b, 0, i);
		}
        bo.flush();
````

字节流读取read（）方法返回的是一个int类型，需要转化为char（通过ASCII码转换）
字节流读取readline（）方法返回的是一个String类型（BufferedReader）

​		3、获取到的字符串使用DataInputStream来读取。需要使用ByteArrayInputStream来获取字符串的字节输入流

````java
		DataInputStream dataInputStream = new DataInputStream(new ByteArrayInputStream("inhaoya".getBytes()));
		while (dataInputStream.available()!=0) {
			System.out.println((char)dataInputStream.readByte());
		}
````

​		4、基本读取文件字符串输出：

1、使用将字符串通过StringReader读取，然后通过bufferedReader进行包装
2、使用fileWriter创建一个文件输出流，再通过BufferedWriter包装，最后使用PrintWriter进行修饰，对输出的字符串进行格式化

		BufferedReader bufferedReader = new BufferedReader(new StringReader("fnsad"));
		BufferedWriter bufferedWriter = new BufferedWriter(new FileWriter(new File("d:\\ni.txt")));
		PrintWriter printWriter = new PrintWriter(bufferedWriter);
		String s;
		while((s=bufferedReader.readLine()) !=  null) {
			printWriter.println(s);
		}
		printWriter.close();
​		5、在java5后，又对printWriter添加了一个辅助构造器，简化了printWriter的修饰创建过程，直接通过file对象来创建一个使用缓冲区、格式化的字符输出流。

````java
    PrintWriter  printWriter  = new PrintWriter(new File("D:\ni.txt")
````

因此对于字符串的输出，直接使用PrintWriter；其他类型数据（如图片），使用缓冲区的字节输出流进行输出

8、使用DataInputStream和DataOutputStream读取数据的好处：

无论读写数据的平台差异有多大，都可以准确的对java基础数据进行读写；

对于java字符串，只能使用UTF-8编码来保证数据读取的准确性，它们提供writeUTF()、readUTF(),来默认使用UTF-8编码读写字符串

9、由于这些数据读写的过程编码非常复杂，因此可以自定义工具类，来定义数据读取、写入的方法，提供I/O流代码的复用

如从文件中读取数据、将数据保存到文件中、读取二进制文件

读取二进制文件：

````java
     FileInputStream fileInputStream = new FileInputStream(new File("D:\\in.txt"));
     BufferedInputStream br = new BufferedInputStream(fileInputStream);
//   根据缓冲流中的所有数据字节数来创建对应的字节数组（available（）获取字节流中的全部字节数）
     byte [] data= new byte[br.available()];
//   将数据读取到data字节数组中
     br.read(data);
````

10、缓冲流在使用时，默认使用8KB大小的缓冲区，当存满时，就进行读写，然后再继续通过IO缓冲数据。read（）方法使用时，缓冲区不够并不会影响其执行；当使用readline（）时，读取的一行数据，可能因为缓冲区不够，而强行截取为一行。

​		对于缓冲输出流，提供一个flush（）方法，防止输出流在将数据缓冲到内存后，只有当缓冲区满后才会将缓冲区的数据写入到文件中，这样就会阻塞；当在read之后加上flush方法，就能将缓冲区的数据清空写入到文件中，之后再关闭流。

close方法执行时，会执行flush（）方法，当之后就不能继续使用该缓冲流；
而flush方法执行后，可以继续使用缓冲流，直到close关闭资源；因此理论上若需要关闭缓冲流时，可以省略flush（）方法

11、标准I/O模型，java提供了System.in、System.out、System.err

System.out、Sytem.err默认被包装成了printStream对象,因此我们可以直接使用system.out、System.err来打印数据

System.in默认是一个未被包装的InputStream，因此需要对System.in进行读取时，需要先对其进行包装转化为BufferedReader对象（这样更好的读取system.in中的数据（字符串））

12、System提供重定向的静态方法：System.setIn（）、System.setOut（）、System.setErr（）；修改system使用的输入流（修改读取的目标）、输出流（修改打印的目标），重定向只能使用字节流。

但记住重定向使用后，需要将其还原初始的状态：System.setOut(console),即还原为控制台打印输出和数据读取

console为自定义变量，保存之前system.out的对象

13、System.in获取控制台输入数据后，会一直处在阻塞状态，等待控制台输入；不会像文件中有EOF终止符来提示readLine==null，因此需要自定义提供一个字符标识，来跳出输入流的读取循环。

14、java1.4引入了新的I/O类库（java.nio.*包），其目的是提供javaI/O的速度。实际上，旧的I/O包也被nio重新实现过，因此使得旧I/O的速度也有了提升，包括文件I/O和网络I/O。

而新I/0速度提示的关键在于,其现在使用的结构更接近于操作系统执行I/O的方式：使用channel（通道）和ByteBuffer（缓冲器）来完成数据传输。channel可以看作为数据源，通过操作缓冲器来对channel进行数据交互（缓冲器获取channel中的数据、缓冲器向channel发送数据）

15、ByteBuffer（缓冲器）是一个基础类，它可以通过方法选择器来读取或输出原始的字节类型和基本数据类型；但是不能输出读取对象，即使是String。

16、旧I/O类库中有三个类被修改，用于产生FileChannel。它们都是字节流，FileInputStream、FileOutputStream、RandomAccessFile。

​		1、使用新I/O类来进行数据写入文件

````java
		FileOutputStream fileOutputStream = new FileOutputStream("data.txt");
		//使用文件输出流来创建filechannel
		FileChannel channel = fileOutputStream.getChannel();
		//将数据写入缓冲器
		ByteBuffer byteBuffer = ByteBuffer.wrap("nihao".getBytes());
		//使用channel将缓冲器中的内容写入文件
		channel.write(byteBuffer);
		channel.close();
````

​		2、使用新I/O类来读取文件数据

````java
FileInputStream fileInputStream = new FileInputStream(new File("D://data.txt"));
		FileChannel channel = fileInputStream.getChannel();
		//创建ByteBuffer对象，需要知道缓冲器大小（使用wrap（）时会根据数据的字节来自动定义大小）
		//也可以使用allocateDirect（）方法获取一个更“直接”的缓冲器，能够达到更快的数据。
		//但是创建时的分配开支也更大，需要根据实际来取舍
		ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
		int len =0;
		//防止缓冲器大小不够，使用循环来将数据全部读取
		while((len=channel.read(byteBuffer)) != -1) {
			//在获取缓冲器数据时，需要执行flip方法，来提示缓冲器做好准备（这样可以获取最大读取速度）
			byteBuffer.flip();
		//按字节逐一读取,转化char拼接为String
		//	while(byteBuffer.hasRemaining()) {
		//		System.out.println((char)byteBuffer.get());
		//	}
            //获取对应数据长度的字节数组，用于字节输出流写入文件
        	byte[] b=new byte[len];
			byteBuffer2.get(b, 0, len);
  			
			//使用完后，若要继续使用该缓冲器时，需要调用clear方法清空缓冲器数据
			byteBuffer.clear();
		}
		channel.close();
````

​		3、通过使用channel的方式，可以简化文件数据转移的过程

````java
		FileChannel
		  inChannel=new FileInputStream("").getChannel(),
		  outChannel=new FileOutputStream("").getChannel();
		//将输入流通道中的数据复制转移到输出流通道中
		inChannel.transferTo(0, inChannel.size(), outChannel);
````

**17、ByteBuffer使用的是字节，在输出打印时，需要进行数据转换：**

ByteBuffer对象提供asCharBuffer等方法，将字节转化为基础数据的视图对象，直接可以打印输出

当ByteBuffer转换为CharBuffer视图时，并不会使用任何编码格式，因此就会出现汉字乱码

在ByteBuffer和其视图对象进行put（）和get（）方法后，需要执行rewind（）将数据指针返回到数据开始处，从而对ByteBuffer进行其他操作；因此ByteBuffer不能进行分段式读取，不能指定空间段读取（这是和数组结构不同之处）

ByteBuffer通过其视图对象将基础数据类型储存到其中（即对视图缓冲器进行添加操作时，会影响缓冲器中的数据），并转化为字节数据，之后再通过不同视图缓冲器进行字节序列解析，翻译成对应基本数据类型；

**18、ByteBuffer索引方法：**

ByteBuffer有四个索引：mark（标记）、postition（位置）、limit（界限）和capacity（容量），可以通过方法获取值，limit（）、capacity（）、postition（）

默认limit和capacity相同，position为0、mark不明确（不存在）
当对于ByteBuffer读取Channel数据后，limit就为数据容量的大小；但使用put（）添加数据时，并不会使limit改变，若超出limit限制则报错内存溢出

当进行ByteBuffer输出或生成视图缓冲器时，只会获取position和limit之间的数据；
而执行get（）或put（）（不指定索引位置）后，会将position向前进一位（根据数据类型所占字节数）；

使用带索引参数的get（）、put（）并不会修改position的值

rewind（）方法可以重置position值0，并且此时mark值会消失

mark用于记录某时的position，通过mark（）方法记录，然后通过reset（）方法，使position转变为mark的值

clear（）清空缓冲器，将position值设置为0，limit为容量值（capacity）

通过以上结论可以得出：

1、flip()方法在ByteBuffer被读取前执行，可以加快读取速度。原因：

​	hasRemaining（）是查找介于position和limit之间的元素；而flip（）是将limit的值设置为和postiton值相等、postion的值设置为0，因此hasRemaining在判断时，只会判断position此时的一个值。而使用get（）方法后，会将position的值增加，limit随之和position一起变化。最后直到hasRemaining（）判断为flase，limit就不会在变化。

2、在执行ByteBuffer输出或生成视图解析器时，需要先执行rewind方法，才能获取全部数据

3、在使用ByteBuffer读取通道（Channel）数据后，limit的值会变为数据的容量，因此在使用put继续添加数据时，会出现内存溢出异常，需要调整limit的值。

**19、nio的内存映射文件**

​		内存映射文件允许创建修改一些因太大不能放入内存中的文件。使用内存映射文件后，可以假定文件已经放入再内存中。

javaI/O类库除了读、写流，还要一个自我独立的类：RandomAccessFile；用该对象直接对文件进行读写操作。

内存映射文件通过RandomAccessFile来进行内存映射文件：

````java
		//通过RandomAccessFile创建文件通道，并指定RandomAccessFile操作文件的方式：r只读、w只写、rw读写
		 FileChannel channel = new RandomAccessFile("D:\\data.txt","rw").getChannel();
		//在通过fileChannel来创建文件的内存映射（即一个映射缓冲器），并指定操作文件通道数据的方法（读+写）、映射文件内容索引初始位置、缓冲器容量大小
		 MappedByteBuffer mbb = channel.map(FileChannel.MapMode.READ_WRITE, 0, 1024);
        //通过put（）将数据写入到当前位置；也可以使用put（index）指定要写入的文件位置
		 mbb.put((byte)'x');
		//通过get（）读取当前位置的数据；也可以使用get（index）指定要读取的文件位置
		 mbb.get();
````

**性能对比：**

虽然建立映射文件的花费很大，但整体速度还是比使用I/O流更加快

并且使用该方式就代替了RandomAccessFile的单独使用

20、在Java1.4中引入了文件加锁机制，保证某个共享资源文件在一个时间类只能被一个线程操作。

​		                              通过对fileChannel调用tryLock（）或lock（）；tryLock（）为非阻塞式，当文件被其他线程占用并持有其文件锁时，就直接返回null；lock（）为阻塞锁，会一致阻塞线程来获取锁。

对于SocketChannel、DatagramChannel和ServerSocketChannel不需要加锁，因为它们是从单线程实体继承而来。我们通常不在多进程间共享网络Socket。

21、JAVAI/O类库支持读写压缩格式的数据流，它们继承了字节流的层次结构。

ZipInputStream、ZipOutputStream      使用zip压缩算法读取数据（还是字节类型）

GZIPInputStream、GZIPOutputStream    使用GZIP压缩算法解压数据（还是字节类型）

它们的基类是DeflaterOutputStream、InflaterInputStream

 		1、使用GZIP进行简单压缩

````java
			BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(new GZIPOutputStream(new FileOutputStream("data1.gz")));
			BufferedInputStream bufferedInputStream = new BufferedInputStream(new FileInputStream(new File("D:\\data.txt")));
			System.out.println("开始压缩数据");
		    int i;
		    //由于压缩输出流只能操作字节，因此只能使用字节输入流读取字节数据（使用字符流读取到的字节数据压缩会出错）
		    while((i=bufferedInputStream.read()) != -1) {
		    	//通过包装过的GZIPOutputStream来压缩数据，并写入文件
		    	bufferedOutputStream.write(i);
		    }
			bufferedOutputStream.write("sfadfsadfsafa".getBytes());
		    System.out.println("完成压缩并生成压缩文件");
		    bufferedOutputStream.close();
		    bufferedInputStream.close();
````

​			2、读取压缩文件

````java
BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(new GZIPInputStream(new FileInputStream(new File("data.gz")))));
		String s;
		while ((s=bufferedReader.readLine()) != null) {
			System.out.println(s);
		}
````

**使用GZIP压缩算法，只能压缩单个文件（单个数据流）**

**而ZIP压缩算法可以对多文件（多数据流）进行压缩：**

1、ZIP文件压缩

````java
		 BufferedInputStream bis = new BufferedInputStream(new FileInputStream(new File("D:\\data.txt"))); 
		 FileOutputStream fos = new FileOutputStream(new File("mmm.zip"));
		 //添加数据校验和输出流，包装流中数据的完整性（校验方式有两种Adler32（更快）、CRC32（更准确），zip压缩使用的ZipEntry只支持CRC32数据和校验，）
		 CheckedOutputStream cos = new CheckedOutputStream(fos, new CRC32());
		 //包装zip压缩流       
		 ZipOutputStream zos = new ZipOutputStream(cos);
		 //添加zip压缩信息（压缩文件可利用的数据）
		 ZipEntry zipEntry = new ZipEntry("");
		 zos.putNextEntry(zipEntry);
		 int c;
		 while ((c=bis.read()) != -1) {
			 zos.write(c);
		 }
		 bis.close();
		 zos.close();
````

3、ZIP格式文件解压

````java
FileInputStream fis = new FileInputStream(new File("mmm.zip"));
		//使用CRC32进行校验和
		CheckedInputStream cis = new CheckedInputStream(fis, new CRC32());
		ZipInputStream zis = new ZipInputStream(cis);
		BufferedReader br = new BufferedReader(new InputStreamReader(zis));
		
		ZipEntry zipEntry;
		//循环遍历每一个压缩文件，并解压获取其压缩信息和文件内容
		while((zipEntry=zis.getNextEntry())!=null) {
			System.out.println(zipEntry);
			String s;
			s=br.readLine();
			System.out.println(s);
		}
````

除了创建zip输入流来解压zip格式文件获取压缩信息（ZipEntry）外，还要一个更简单的方法：

使用ZipFile对象读取压缩文件的ZipEntry

22、jar包也使用了类似ZIP格式，将一组文件压缩到一个单一压缩文件中。但是jar打包工具没有zip工具强大，并不能对jar包中的文件进行更新操作，只能从头创建jar包。

23、什么是对象序列化：

对于java对象，当我们需要保存该对象信息时，只能将其拥有的信息保存起来，并不能完整的保存要给对象，这样就违反了java面向对象的原则。因此java通过对象序列化，来将实现了Serializable接口的对象转化为一个字节序列，之后可以通过反序列化的方式返回一个完全相同的对象。实现对象信息的持久化。

**我们可以通过对象序列化来完成对象的深度复制**

**JAVA提供了对象流来完成对象的序列化和反序列化过程（前提时该对象类实现了Serializable接口）：**

通过ObjectOutputStream（对象输出流）的WriteObject（）方法来对对象进行序列化，生成字节数组

通过ObjectInputStream（对象输入流）的ReadObject（）方法来对对象字节数据进行反序列化，
生成一个Object对象

24、对序列化进行控制：

​	1、继承Externalizable类来代替Serializable接口,此时对象的任何东西都不可以自动序列化，之后再通过writeExternal（）内部对需要序列化的内容进行标记

​	2、使用transient（瞬时）关键字，可以对某个成员变量（不能修饰方法）关闭序列化，如对象中保存的密码，我们不希望被序列化。

25、XML：

对象序列化只是java的解决方案：只有java程序才能反序列化该对象。因此有了一种更具有操作性的解决方案：将数据转化为XML格式，从而被其他平台、语言使用。

java.xml.*类库提供到了java对象序列化到XML中的方法。

26、java还提供Preferences API来将对象持久化到系统中。它本来是用于存储和读取用户的偏好和程序配置项设置。根据操作系统的不同，资源保存的地方也不同；比如WINDOWS时使用注册表来保存。

## 枚举类型

1、枚举enum的作用：将一组具名的值的有限集合创建为一种新类型，并作为常规程序组件使用。

​	  **创建enum的方式：**

​			1、在一个类外，直接通过enum关键字创建枚举；编译器会自动生成一个相关类，并默认继承java.lang.Enum；且该enum只能使用default  权限修饰符，只能在同包类中使用

````java
enum Signal {  
    GREEN, YELLOW, RED  
}  
````

​			2、编写一个enum类，默认继承java.lang.Enum，只能使用public 权限修饰符，完全公开

````java
public enum Signal {
    GREEN,YELLOW,RED
}		
````

2、通过使用enum可以代替常量定义，并且在switch中语句中使用，增加代码的可读性（开发者可以通过enum清晰的看到case匹配条件的含义）

​		**switch语句在JDK1.6中支持byte、short、char、int；1.7后才支持string**

​		**switch中的default语句不是必须的，但当case中出现return，并且switch结束后就方法结束时，就必须添加上default：防止switch没有匹配的，导致方法无法返回值。**

3、enum的常用方法：**分为枚举和枚举成员（即枚举实例）**

````java
//通过枚举的values获取该类型枚举的所有枚举成员数组（该方法不是继承Enum类获得的，而是编译器提供的静态方法，因此无法获取其源代码）
Signal[] values = Signal.values();
for (Signal s :values) {
    //获取该枚举实例的声明次序（int值，从0开始）
    System.out.println(s.ordinal());
    //获取该枚举实例所对应的枚举类的calss对象
	System.out.println(s.getDeclaringClass());
    //以String类型获取该枚举实例的值
	System.out.println(s.name());
}

````

4、enum的枚举成员是该枚举类型的一个实例，因此其名字相当于一个变量名，只能使用$、_和字母开头

5、通过对每个枚举成员来添加属性、方法来使枚举代替常量使用：

````java
public enum Wekkk{One(1,"菠菜") {
    //可以单独在某个枚举成员中添加方法，但可以被外部Wekkk类中的同名方法重写（toString方法默认也写在每个enum成员中，因此可以通过在wekkk内中重写所有枚举成员toString方法）
    int getValue（）{
        system.out.printl("这是菠菜")
    }
}
    ,two(2，"黄瓜"),THREE(3，"土豆");
	private int value;
    private String name;              
//  构造方法是为了定义enum类中的枚举成员，并且不能通过代码改变，因此为private；
//	其中的参数需要和枚举成员中的属性一一对应
	private Wekkk(int value，String name) {
		this.value=value;
        this.name=name;
	}
    
    public  int getValue(){
        return value;
    }            
    public String getName(){
        return name;
    }              
}
````

这样通过枚举来代替常量使用，可以更加清晰的反应每个枚举成员所代表的值和含义，增加代码的可读性

6、由于所有枚举类都继承了java.lang.Enum类，java不支持多继承，因此枚举只能实现接口

​		1、通过实现接口，可以添加枚举实例的方法

​		2、在一个接口中，定义不同枚举类实现该接口，来向上转型，让所有枚举实例都为同一个接口类型（来满足现实问题：食物分为水果、大米；水果又分为苹果、香蕉）

7、枚举类的枚举成员在使用时，可以类似看作成枚举类中的静态成员内部类，被static修饰，直接被 枚举类名.枚举成员名  的方式获取枚举成员。且不提供public的构造方法来修改枚举成员属性  或添加枚举成员

8、枚举类所继承的java.lang.Enum类，实现了Comparable接口，具有compareTo（）方法；实现了Serializable接口，可以进行序列化

9、枚举可以通过import startic  com.main.week.*  来静态导入enum。这样enum实例就可以直接通过enum成员名来引入，无需使用enum类型名来修饰。但这样会导致代码可读性降低

10、EnumSet的使用

为什么需要EnumSet：对于Enum，它的值都是唯一的，但是却不能够删除、添加元素，大大减少了enum的使用性，因此在java 5中引入了EnumSet

​	1、EnumSet区别于传统set集合，内部实现没有使用常见的数据结构（数组、链表、哈希表、红黑树），而是使用的位运算来进行集合的基本操作；因此EnumSet与set集合相比，又更快的速度。
​	2、EnumSet元素必须来自于一个enum，并且其中的元素排序根据enum中的成员声明次序来决定；并且不能添加null
​	3、EnumSet常用方法：

````java
		//EnumSet没有暴露的构造方法，只能使用该静态方法来创建实例		
		//1、创建一个空的week枚举类型的EnumSet
		EnumSet<Week> enumSet = EnumSet.noneOf(Week.class);
		//2、创建一个接受了week类型参数的EnumSet(这里有多个重载方法，支持1、2、3、4、5、可变参数)
		EnumSet<Week> enumSet1 = EnumSet.of(ONE);
		//3、创建一个接受week枚举部分参数的EnumSet（按照Enum次序获得该区间段的所有成员）
		EnumSet<Week> enumSet2 =EnumSet.range(ONE, DFDSF);
		//此时使用了枚举静态导入，这里可以减少代码累赘（不用重复写Week类型修饰）
		//添加枚举实例
		enumSet.add(ONE);
		//添加enumSet1中的所有元素
		enumSet.addAll(enumSet1);
		//删除枚举实例
		enumSet.remove(ONE);
		enumSet.removeAll(enumSet1);
		//获得两个enumSet的并集
		EnumSet<Week> complementOf = EnumSet.complementOf(enumSet1);
````

11、EnumMap的使用

在java 5中引入，以同一个enum成员为键，value可以为任何类型。因此EnumMap的key不能为null；并且EnumMap中元素的排序和key在enum的次序有关。

创建方式为：

````java
EnumMap<Week, String> enumMap = new EnumMap<Week, String>(Week.class);
````

和EnumSet不同，EnumMap提供公开的构造方法来new获取实例，并指定key对应的enum类型。之后其他操作和普通map没有区别。主要作用是，配合Enum来实现map数据结构的功能。

12、多路分发，以两路分发为例，就是当出现两个类型向上转型为同一类型后，通过对其类型的判断来执行不同的方法。如石头剪刀布，判两者的类型，从而得出结果。在其中enum就可以作其类型，属性作为结果，增加方法代码的重用

## 注解

1、java注解：

​		注解，也被称为元数据，为在代码中添加数据信息提供了一种形式化的方法，使我们能在代码执行的某个时刻方便地使用这些数据。

​		注解在JAVA 5中引入，将注解添加在类、成员变量、方法、局部变量和方法参数，从而在代码执行时，执行一些逻辑代码，使用注解中的数据信息，来实现某些常用操作。

作用：使常用功能代码被复用，并且复用代码使用更加简洁、清晰；

​			代替之前配置文件的方式来获取一些必要的数据信息；

​			对于一些逻辑、数据进行编译期类型检查。

2、JDK 5内置三种注解，在java.lang包中使用：

@Override，表示方法为对超类方法的重写，必须方法名与超类方法一致。否则编译器发出错误提示

@Deprecated，表示该方法已过时，当在其他代码调用时，编译器会发出警告

@SuppressWarnings， 关闭不当的编译器警告信息，如强行使用过时方法，就需要添加注解	SuppressWarnings（“deprecated”）	

java1.7 加入 @SafeVarargs，参数安全类型注解，告诉开发者不要对参数做一些不安全操作，否则会被编译器阻止

java 1.8加入 @FunctionalInterface ，函数式接口注解，用于注解接口中的方法可以进行函数式编程，即方法可以转换为lambda表达式

3、java还提供四种元注解，用于创建新的注解（自定义注解），从而使用注解来简化、自动化一些重复性工作。

@Target  定义注解的作用域

@Rectetion  定义注解的生命周期     SOURCE（只在源码显示，编译时丢弃）,CLASS（编译时记录到class中，运行时忽略）,RUNTIME（运行时存在，可以通过反射读取）

@Inherited   注解会随着类的继承而作用到另一个类上

@Documented   生成javadoc

@Repeatable   JAVA1.8新加入的，该注解可以在一个地方使用多个

4、注解的定义

注解通过@interface 关键字来定义，类似于接口的定义

````java
@Target(ElementType.TYPE)  //注解在类、接口、枚举
@Retention(RetentionPolicy.RUNTIME) //运行时生效
public @interface TestUser {
	//定义注解中的成员变量（注解中没有方法）
//	通过 "无参方法" 的形式，来定义成员变量；此时该注解就有id、msg两个属性
	int id();
	//可以通过default关键字来指定该注解属性的默认值
	String  msg()  default "成功";
}
````

注解的成员变量类型可以是：所有基本数据类型、String、Class、enum、Annotation、数组

注解本身是不能进行继承、实现其他注解的

5、注解的实现

**注解本身只能通过注释的方式获取元数据，但对于数据的处理还需要注解处理器来实现注解的真正功能**

1、原生注解处理器的编写：

````java
	public static void main(String[] args) throws Exception {
		//遍历代码中的所有类名，通过java反射机制来获取它们的Class对象
		for (String className : args) {
			Class<?> cl = Class.forName(className);
			//通过Class对象来获取类中的所有信息，包括注解；从而来对注解中的信息、和类的信息进行处理
			TestUser annotation = cl.getAnnotation(TestUser.class);
			if (annotation==null) {
				System.out.println("该类没有@TestUser注解");
				continue;  //直接进行下一次循环
			}
			
			annotation.id();
			annotation.msg();
		
		}
````

对于注解的使用，需要通过遍历所有类的Class对象来扫描获取注解对象（确定该类使用了对应的注解），执行对应的注解处理器。并且反射机制执行效率低，因此注解使用时的时间成本较高。

2、由于原生编写注解处理器的方式需要遍历所有类，且反射慢。因此java提供了要给apt工具，来专门编写注解处理器。

APT（AnnotationProcessorFactory  注解处理工厂）工具，不是使用的java反射机制，而是通过操作源代码来获取类信息，而不是通过反射读取编译后的calss文件。并且可以直接指定需要处理的类，而不需要遍历所有类。

## 并发

1、基本线程机制：

通过将程序划分成独立运行的任务，然后将任务交给线程执行。使用多线程机制，来将任务并发执行。

​	1、创建runnable接口的实现类，在重写的run（）方法种编写任务代码

````java
public class YhTask implements Runnable{
	
	private  int countDown =2;
	private static  int  count = 0;
	private final int id = count++;
	
	public YhTask(int countDown) {
		this.countDown=countDown;
	}	
	
	@Override
	public void run() {
		while (countDown-- >0) {
			System.out.println("#"+id+"("+(countDown>0?countDown:"发射")+")");
		}
    }
````

​	2、在主函数main（）方法中，通过task类的实例化对象创建Thread线程类对象，并调用start（）方法

````java
public static void main(String[] args) {
    new Thread(new YhTask(8)).start();
}
````

**start（）方法会执行Thread线程对象初始化操作，启动线程。之后该线程调用task类的run（）方法，执行任务。但start（）方法Thread线程启动后，直接返回。因为运行start（）方法的线程（main方法线程）和执行任务的线程为两个不同线程**

**在创建Thread对象时，即使没有使用引用（变量接收），也会自动创建一个引用，直到线程被回收**

2、在java 5时，添加了java.util.concurrent包，提供了执行器类（Executor）来管理线程类（Thread），简化并发编程。Executor代替了显式创建Thread类，通过ExecutorService（执行任务服务，具有生命周期的Executor，即线程池）来执行runnable对象任务

````java
	ExecutorService executorService1 = Executors.newSingleThreadExecutor();
	executorService1.submit(new YhTask(9));
		
	ExecutorService executorService2 = Executors.newFixedThreadPool(2);
	executorService2.submit(new YhTask(10));
		
	ExecutorService executorService3 = Executors.newCachedThreadPool();
	executorService3.submit(new YhTask(11));
````

Executors类（执行器工厂）提供了三种创建ExecutorService对象的静态方法：

1、CachedThreadPool  缓存线程池  不手动指定最大线程数maximumPoolSize，而是根据系统资源大小来自动设定。corePoolSize默认为0；当指定任务后，没有多余空闲生命周期线程时，创建新线程。当任务数大于最大线程数时，任务进入SynchronousQueue队列进行排队执行。

**CachedThreadPool的线程空闲生命周期（keepAliveTime ）默认为60s，当执行完任务后，60s内无任务，则线程被回收**

2、FixedThreadPool  固定线程池 指定可创建的线程数（corePoolSize，maximumPoolSize为指定值），一个线程执行一个任务，当任务数大于线程数时，任务放入 LinkedBlockingQueue<Runnable>队列，进行排队执行

3、SingleThreadPool  单一线程池    只用一个线程来执行任务，若提交多个任务，则使用同一个线程按照任务提交顺序，放入 LinkedBlockingQueue<Runnable>队列，排队依次执行。 SingleThreadPool 线程池可以看为corePoolSize=1（最小线程数），maximumPoolSize=1（最大线程数）的FixedThreadPool 线程池

**FixedThreadPool和SingleThreadPool  的线程不会被自动回收（keepAliveTime 无效），只有当线程池被关闭时，才会在任务结束后回收**

3、三种基本线程池选择：

CachedThreadPool 线程池中的线程都会因为超时而被回收，所以几乎不会占用什么系统资源，比较适合任务量大但耗时少的任务

FixedThreadPool 用于控制多线程的最大并发数，适用于任务量比较固定但耗时长的任务。

SingleThreadPool  只使用一个线程，适用于多个任务顺序执行的场景。

4、带返回值的线程任务：

Runnable接口时执行独立任务，但没有返回值

Java5引入了Callable接口，其call（）方法（对应run（））返回一个泛型返回值

````java
public class PpTask implements Callable<String>{
	@Override
	public String call() throws Exception {
		System.out.println("发射");
		return "成功";
	}
}
````

该任务只能使用submit（）方法进行执行，并且返回一个Future对象，用于提供对call方法结果的操作：

Futrue.isDone()  用于判断Future对象是否初始化完成，即任务是否执行完成，call（）方法返回返回值

Futrue.get()  用于获取Futrue对象中保存的call（）方法返回值，不带参的get（）方法会一直阻塞，指定任务完成返回返回值；而带参的get（）方法可以设置等待时间（前则为时间数，后者为时间单位），若时间内任务还未执行完，则返回null。

Futrue.cancel(true)  用于单独调用线程池中对应线程任务的Interrupte（）中断方法

````java
		ExecutorService service = Executors.newSingleThreadExecutor();
		Future<String> submit = service.submit(new PpTask());
		System.out.println(submit.isDone());
		String string ="";
			try {
				string = submit.get(1000, TimeUnit.MILLISECONDS);
			} catch (Exception e) {
				e.printStackTrace();
				string="失败";
			}finally {
				System.out.println(string);
			}
````



5、线程池（ExecutorService）的常用方法

1、execute（）  执行（提交）任务，只能执行runnable接口任务
2、submit（）执行（提交）任务，可以执行runnnale和callable接口任务，并返回callable接口任务返回值的参数化Future对象    
3、shutdown（）关闭线程池，不能继续进行使用该线程池对象进行任务执行，其他任务全部执行完后，该对象被回收

4、shutdownnow（）调用线程池中所有线程的interrupt（）中断方法

 **推荐线程池使用submit（）方法执行任务，可兼容两种任务接口，原因是 ：**

1、可以获取Callable接口任务的返回值，判断任务是否执行成功

2、对于runnale任务，异常只能进try-catch内部处理，而不能像Callable任务默认抛出异常，通过对返回的Future.get()方法进行捕获，处理异常。

实际需要场景：如同时执行多个task，希望一个task发生异常后，其他task也停止执行。这样就需要task能够throws抛出异常。

6、后台线程

后台线程是指在程序运行中后台指提供一些通用服务的线程，并不属于程序不可或缺的一部分

执行main（）方法的线程为非后台线程，当非后台线程全部关闭时，程序终止，并关闭所有后台线程

通过thread.setDaemon（true）方法，将线程设置为后台线程；thread.isDaemon（）来进行判断

**后台线程在run（）方法中，不会执行finally{}中的代码，直接结束run（）方法**

7、线程优先级

线程调度器会倾向于让优先级更高的线程先执行，但不意味者优先级低的不会得到执行，只是优先级低的线程执行频率较低。

**因此线程优先级不会导致死锁，我们也不能通过操作线程优先级来控制任务的执行顺序**

绝大部分时间，我们都是使用线程的默认优先级执行，可以通过thread.getPriority（）、setPriority（）来获取、修改线程的优先级属性

8、线程让步和睡眠

**线程让步，使用Thread.yield（）是将线程强行转变为就绪状态，并将此次cpu时间片段转移给优先级相同或更高的线程竞争，因此可能存在自己又变为运行状态**

线程让步的实际场景：当循环执行任务时，让同优先级的每个任务执行一次的cpu片段时间更加均匀，但实际中线程控制基本不需要使用让步

**线程睡眠，使用Thread.sleep（）是将线程强行转变为阻塞状态。只有经过指定时间后，才能转变了就绪状态**

线程睡眠相比让步，对于多线程的执行控制使用性更强。可以有效控制线程的执行时间（每多久执行一次任务）

​	1、sleep（）方法可能抛出InterruptedException异常，需要在方法中进行处理

​	2、在java5后，提供了另要给sleep（）版本：TimeUnit.MICROSECONDS.sleep(2)  通过TimeUnit类，来显示选择sleep（）中的时间单位。使代码有更好的可阅读性

9、自定义ThreadFactory（线程工厂对象）

**通过实现ThreadFactory接口，重写newThread（Runnable r） 方法，以工厂模式通过task创建线程：**

​	此时可以通过在newThread方法中添加代码，工厂化定义Thread对象的属性，如是否为后台线程、线程名、线程优先级。。。

````java
public class MyThreadFactory implements ThreadFactory{
	@Override
	public Thread newThread(Runnable r) {
		Thread thread = new Thread(r);
		thread.setName("nihao");
		thread.setPriority(2);
		thread.setDaemon(true);
		return thread;
	}
}
````

10、线程创建的其他方式

1、通过继承Thread类，直接为线程对象编写run（）方法

````java
public class YhThread extends Thread{
	@Override
	public void run() {
		super.run();
		System.out.println("发射");
	}
}
````

此方式一般用于简单任务，定义一些成员变量，直接用该类进行测试，添加main方法，调用该类对象（实际代码中，一般创建task层，便于代码层次清晰）

2、也可以直接new Thread对象，使用匿名内部类的方式，重写run（）方法

````java
		Thread thread = new Thread("Thread1") {//可以通过Thread构造方法来设置该线程对象的线程名
			@Override
			public void run() {
			}
		}.start();
````

该方法是直接在声明Thread对象时，添加该线程的任务执行代码，使得代码更加整体，减少类之间的糅合，方便代码阅读和维护

11、在run（）方法中设置执行该任务的线程对象属性

**Thread.currentThread（）  用于调用该线程的引用，从而调用线程对象方法（非静态方法）**

当在任务run（）方法中需要设置对于线程Thread对象的属性时，可以通过该静态方法，获取执行该任务的线程对象的引用，从而调用其它非静态方法

12、线程运行同步

通过在main（）方法（主函数线程）或者task的run（）方法中调用另一个线程的join方法，使得主函数线程（或自定义线程）必须等待调用join（）方法的线程执行完后，才能同步执行

````java
	public static void main(String[] args)  {
		Thread thread = new Thread(new YhTask(10));
		thread.start();
		try {
			thread.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("结束");
	}
	}
````

这样可以使一个线程在另一个线程中使用时，实现同步，使两线程不再并发执行

join（）方法必须在start（）方法后执行，即只有线程启动后才能进行同步；否则没有效果

join（）可能抛出InterruptedException异常（如对应线程在运行调用interrupt（）中断方法），需要使用try-catch

**join方法同步实现原理：**

join（）方法实际上是循环执行一个Object的wait（）方法（使调用该方法的线程（即执行thread.join（）方法代码线程，如主线程）进入阻塞状态），直到当前线程（调用join（）方法的线程）isAlive（）方法返回flase（即线程生命周期结束Dead）

13、线程的异常捕获

由于线程的并发执行的，因此并不能在某个线程中（主函数）去捕获另一个线程中抛出的异常。在java5之前使用线程组方式捕获

在java5后，提供Thread.UncaughtExceptionHandler接口来定义线程的异常处理器

````java
//定义线程异常处理器
public class MyExceptionHander implements UncaughtExceptionHandler{
	@Override
	public void uncaughtException(Thread t, Throwable e) {
		System.out.println(t.getName()+"线程出现了问题");
	}
}


public static void main(String[] args)  {
    	//设置单个线程的异常处理器
		Thread thread = new Thread(new ExceptionTask());
		thread.setUncaughtExceptionHandler(new MyExceptionHander());
		thread.start();
		
		//设置所有线程的默认异常处理器
		Thread.setDefaultUncaughtExceptionHandler(new MyExceptionHander());
    	//通过线程池或者thread.start()启动任务
		ExecutorService executor = Executors.newSingleThreadExecutor();
		executor.execute(new ExceptionTask());
    	new Thread(new ExceptionTask()).start();
		
    	//使用submit来提交任务
    	Future<?> submit = executor.submit(new ExceptionTask());
		try {
			System.out.println(submit.get());
		} catch (InterruptedException | ExecutionException e) {
			e.printStackTrace();
			System.out.println("发生异常");
		}
	}
````

线程池的execute方法可以直接使用异常处理器捕获处理异常，该异常处理器也叫做未捕捉异常处理器（UncaughtExceptionHandler），用于处理我们在runnable方法中忽略的异常，防止直接使线程宕掉。

而submit方法本身将异常封装在返回的Future对象中，当使用该对象时，随着java.util.concurrent.ExecutionException异常一起抛出，并不会被异常处理器处理

15、解决多线程共享资源竞争

**1、使用synchronized（同步）关键字**

​		synchronzied可以修饰方法或者代码块，当修饰的方法或代码将要被执行时，会先检查锁是否可用，然后获取锁，执行代码，最后释放锁。

​	**1.1、使用synchronized方法**

​		一般共享资源都是类的一个属性，通过synchronized方法进行资源使用。未防止资源被其他线程直接调用，一定要对共享资源添加private 关键字修饰

​		所有对象都含有一个单一的锁（也称为监视器），当调用该对象的synchronized方法时，该对象就会被加锁，其对象的其他所有synchronized方法都需要该方法被执行完后，释放对象锁才能被调用

​		一个任务可以获得多个对象锁（包括重复对象和不同对象）。jvm会追踪对象被加锁的次数，只有当加锁计数为0时，才能被其他线程使用该对象的所有synchronized方法

​		当调用synchronzied修饰的static方法时，获取的是类锁。用于在类的范围内防止static数据的并发访问。类锁和对象锁并不会互相影响

​	**1.2、使用synchronized代码块（同步控制块）**

​		当使用synchronzied修饰的代码块时，其内置锁需要进行指定；可以指定对象锁或者类锁。

````java
	public void setA1() {
		synchronized(this){//获取调用该方法的对象锁，此时该对象的所有synchronzied方法和指定该对象的代码块都不能被其他线程调用；
			a=a-1;
			a=a+5;
		}
	}

	public void setA1() {
		synchronized(this.getClass()){//获取该方法所在类的类锁，此时该类的所有synchronzied静态方法和指定该class对象为内置锁的代码块都不能被其他线程调用；
			a=a-1;
			a=a+5;
		}
	}

	private YH yh;
	public void setA(yh){//获取该类中的yh变量的对象锁，此时该yh变量的所有synchronzied方法和指定该变量为内置锁的代码块都不能被其他线程使用；
        synchronized(yh){
            
        }
    }
````

由这三种同步控制块的使用可以知道，同步控制块比synchronzied方法更加灵活：

1、减少不必要的同步代码，使其他线程在有限时间内有更多地访问（在安全情况下）

2、可以指定需要的内置锁，从而在同步该代码块的同时，同步内置锁对象的synchronzied修饰的方法（根据内置锁的对象不同而改变）。满足该方法在执行时，其他类中的方法（被synchronzied修饰）调用会被阻塞

**2、使用显式的Lock对象**

java5 的java.util.concurrent类库提供的Lock对象，来显式的创建和释放锁。它与使用synchronzied关键字的内建锁的形式相比，代码缺乏优雅性，但使用更加灵活。

````java
		
    private  ReentrantLock lock = new ReentrantLock();
	@Override
    public void run() {
		lock.lock();
		try {
			while (countDown-- >0) {
				System.out.println("#"+id+"("+(countDown>0?countDown:"发射")+")");
			}
		} finally {
			lock.unlock();
		}
    }
````

通过lock（）、unlock（）方法来获取锁和释放锁。相比于使用synchronized关键词，写的代码量更多，用户错误出现的可能性也就越高。因此只有在特殊情况下，才使用显示的lock对象，进行更细粒度的同步控制。

16、java多线程的原子性和可视性

多线程的原子性指：对于某些具有原子性的操作，在多线程中，从执行开始，必须执行完成后才能进行上下文切换（多线程切换）。如对于读写除long、double之外的基本数据类型、控制台打印等。

具有原子性的操作是由线程机制保证其不可中断的，因此该操作不需要进行加锁同步。但利用原子性并不能代替并发中同步的能力，它存在许多致命的问题。如在多处理器系统中，针对于某个CPU，该原子操作是不可中断的，但它还是可以被其他CPU进行多线程的并发使用。

**因此对于具有原子性的操作多线程并发执行时，要保证操作中的变量具有可视性**

对于多线程执行同一个共享资源变量操作时，变量操作的结果时暂时性保存在cpu的本地缓存中的，不会马上修改主存（物理内存）中的变量值。因此就会导致可视性不能保证

可视性：就是保证共享资源在并发使用时，直接在主存中操作，而不是使用速度更快的cpu缓存。这样每一个线程的CPU本地缓存在操作资源时，都是当前的最新修改值。

17、Volatile（易变） 关键字 

**java通过Volatile（易变）关键字来声明变量的具有可视性**

当使用同步时，会使变量操作只能一个线程执行完后，才能被其他线程再次执行。并且同步会在释放锁前，将资源变量的最新值刷新到主存中。**因此同步也保证了变量具有可视性，只是通过避免并发访问该资源的场景的发生来实现。**

**Volatile关键字实现变量可视性，但不能保证变量操作具有原子性**。因此当执行n++非原子操作时，会出现两个线程同时获取到相同n值，并执行返回相同的值。使两次操作转化为了一次操作。

看起来像：

````java
n=5;
a=n+1;
b=n+1;
n=a
n=b
n=6
````

**因此Volatile关键字需要配合原子操作使用。一般在多线程时，修饰共享变量，保证变量的读写操作是线程安全的。并且Volatile并不会使线程阻塞，相比于同步锁效率更高**

**其他情况下，都需要使用同步锁。**

18、线程的生命周期

新建（New），在线程被创建时，短暂处于此状态，此时它已经分配了内存空间并初始化。等待start（）方法调用，线程调度器转变其生命周期状态

就绪（Runnable），线程处于可执行状态，等待调度器分配cpu时间片段，进行任务执行

运行（Running），线程获地cpu地使用权，任务代码被执行

阻塞（Blocked），由于某种情况，使调度器忽略该线程，不分配cpu时间片段，直到线程重新进入就绪状态。

死亡（Dead）：线程不能被调度，一般在run（）方法返回时转变为dead，但此时线程还是可以被中断，线程对象并没有被回收。

进入阻塞状态的常见原因：

1、线程任务执行调用了sleep（）方法

2、调用了wait（）方法

3、任务等待输入输出完成

4、该任务调用了的同步方法，并且该方法对应的锁不可用

19、线程中断

suspend（）      使线程暂停执行，不退出可执行态（就绪状态）
resume（）       将暂停的线程继续执行

这个两个方法已过时，因为会导致死锁

stop（）使线程强行中断，直接方法也已过时，因为它会破坏线程的同步的原子性，太过于暴力不推荐

线程中断常用方法：

1、在run（）方法中提供一个while循环判断条件作为标记，并提供一个公开的方法对其进行修改

当我们需要线程中断时，可以通过方法时标记变为false，从而任务自动完成返回，最后结束线程

**适用于线程不会发生长时间阻塞，从而无法执行执行run方法中的标记**

2、使用线程内部的中断状态作为while循环判断条件

interrupt（）使线程标记为中断状态，并不会中断线程；

1、当线程处于sleep、wait阻塞状态时，会立即退出阻塞状态，并抛出InterrruptedException异常。
2、当线程处于新I/O（使用channels）阻塞状态时，会立即退出阻塞状态，并抛出ClosedByInterruptException异常

此时我们可以通过try-catch来退出循环，结束线程；在抛出异常后，线程将处于非中断状态

isInterrupt（）返回该线程是否为标记为中断状态（但不清除中断状态）

interrupt（） 返回该线程是否为标记为中断状态（会清除中断状态）

**当线程标记为中断状态后，再进入阻塞状态（sleep、wait）时，会直接抛出InterrruptedException异常。**

**这种方式既可以中断普通任务线程，又可以中断处于阻塞状态（将要阻塞状态）的线程**

3、对于因获取synchronized方法的对象锁而处于的阻塞状态和执行普通I/O处于的阻塞状态，并不能通过interrupt（）方法来退出阻塞状态。**只能等到操作执行完后，判断循环结束来中断线程。**

20、多线程在同步中的切换

有时我们需要被同步的代码，此时被其他线程使用（使目前调用该方法的线程释放同步锁）

通过使用wait（）、notify（）、notifyAll（）来进行切换：

这些方法都属于Object类中的方法，它们对应的对象应该需要一致

wait（）执行该方法的线程处于阻塞状态（挂起），并释放调用该方法对象的锁（该线程必须获取了该对象锁）

notify（）使一个处于因释放对应对象锁而阻塞的线程进入就绪状态（唤醒）（当知道只有一个使用该对象锁的线程被挂起）

notifyAll（）使所有处于因释放对应对象锁而阻塞的线程进入就绪状态（该方法更加安全，防止人为遗漏需要唤醒的线程）

由于wait（）、notify（）方法都需要使用同一个对象锁。因此一般使用同步控制块来编写同步代码，从而更加方便指定代码的同步对象锁

````java
private static String a =new String("ds");
	public static void main(String[] args)  {
		new Thread() {
			@Override
			public void run() {
				synchronized (a) {
					System.out.println("strat1");
					try {
						//释放a对象锁，线程挂起等待唤醒
						a.wait();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
					System.out.println("end1");
				}
			}
		}.start();
		
		new Thread() {
			@Override
			public void run() {
				synchronized (a) {
					System.out.println("strat2");
					System.out.println("end2");
					//唤醒所有放弃该锁而挂起的线程
					a.notifyAll();
				}
			}
		}.start();
````

21、使用同步锁，发生死锁的四要素:

​	1、存在共享资源，一个变量会被多个线程使用

​	2、资源在同步使用时，会出现线程等待

​	3、资源在使用时，不存在其他线程等待时进行资源抢占的机制

​	4、出现循环等待，及一个线程等待另一个线程结束。。。最后一个线程等待第一个线程结束，最后发生死锁

22、java5 java.util.concurrent类库提供的并发工具类

​	1、CountDownLatch  倒数锁，在一组线程（单个或多个）执行自己的任务时，使所有线程都在运行到任务的某个节点等待；直到满足条件（倒数锁计数为0）时，再对所有线程全部唤醒

````java
	public static void main(String[] args)  {
		//开始-倒数锁
		CountDownLatch start = new CountDownLatch(1);
		//结束-倒数锁
		CountDownLatch end = new CountDownLatch(10);
		
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		
		for (int i = 0; i < 10; i++) {
			 Runnable runnable = new Runnable() {
				public void run() {
						try {
							//等待 开始-倒数锁释放
							start.await();
							//执行一段时间代码（通过线程睡眠来仿真）
							Thread.sleep((int)Math.random()*10000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						}finally {
//							倒数 结束锁（代表该线程的任务完成）
							end.countDown();
						}
				}
			};
			 executorService.submit(runnable);
		}
		
		//当所有任务分配后，倒数开始锁（让所有线程任务同时执行）
		start.countDown();
		System.out.println("任务全部开始");
	    try {
	    	//此时使main主线程等待所有其他线程任务执行完
			end.await();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println("任务全部执行完");
	}
````

**CountDownLatch实际使用场景：使多个线程任务同时并发执行（如仿真多个远动员同时赛跑）**
	2、CyclicBarrier（循环栅栏），它与CountDownLatch  的作用类似，就是可以让一组线程的任务全部在指定节点等待，当该对象使指定数的线程等待后，此时唤醒全部线程。（当线程唤醒后，已指定数清0，因此该对象可以重复进行线程拦截）

````java
public static void main(String[] args)  {
    	//只指定阻拦的线程数
//		CyclicBarrier cyclicBarrier = new CyclicBarrier(10);
        //设置最后一个完成任务的线程，需要外加执行的任务
		CyclicBarrier cyclicBarrier = new CyclicBarrier(10, new Runnable() {@Override
		public void run() {
			System.out.println("全部线程任务结束或开始");
		}
		});
//		
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		for (int i = 0; i < 10; i++) {
			 Runnable runnable = new Runnable() {
				public void run() {
						try {
							//等待所有线程任务都执行到开始点
							//使线程在此等待，并使cyclicBarrier的线程阻拦数加1
							//当阻拦数达到指定值时，唤醒所有线程，并阻拦数清零
							cyclicBarrier.await();
							System.out.println(Thread.currentThread().getName()+"任务开始");
							//执行一段时间代码（通过线程睡眠来仿真）
							Thread.sleep((int)Math.random()*10000);
						} catch (InterruptedException e) {
							e.printStackTrace();
						} catch (BrokenBarrierException e) {
							e.printStackTrace();
						}finally {
							try {
//								cyclicBarrier可以重复使用，等待所有线程任务都结束
								cyclicBarrier.await();
								System.out.println(Thread.currentThread().getName()+"任务结束");
							} catch (InterruptedException | BrokenBarrierException e) {
								e.printStackTrace();
							}
						}
				}
			};
			 executorService.submit(runnable);
		}
	}
````



**CyclicBarrier实际使用场景：使所有线程全部完成一次任务后，再执行下一步操作（如多线程计算后，然后求和）**

​	3、Semaphore  信号量，用于控制特定资源的线程访问数量

````java
public static void main(String[] args)  {
		//指定信号量的限制线程数，开启公正性（等待最久的线程，下一个执行）
		Semaphore semaphore = new Semaphore(2, true);
//		Semaphore semaphore = new Semaphore(2);
		ExecutorService executorService = Executors.newFixedThreadPool(10);
		//执行10个线程任务
		for (int i = 0; i < 10; i++) {
			Runnable runnable = new Runnable() {
				@Override
				public void run() {
					try {
						//该段代码只能同时被2个线程使用，其他线程等待
						//通过acquire（）、release()方法标记需控制访问的代码块的起止点
						semaphore.acquire();
						System.out.println(Thread.currentThread().getName());
						Thread.sleep(3000);
						semaphore.release();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			};
			executorService.submit(runnable);
		}
````

**需要注意的是 Semaphore 只是对资源并发访问的线程数进行监控，并不会保证线程安全。**

**Semaphore 实际使用场景：用于流量控制，限制资源访问的最大并发数**

​	4、Exchanger 交换器，用于两个线程之间进行数据交换

````java
public static void main(String[] args)  {
	ExecutorService executorService = Executors.newCachedThreadPool();
    //通过泛型来指定需要交互数据的类型
	Exchanger<String> exchanger = new Exchanger<String>();
	
	 Runnable runnable1 = new Runnable() {
		public void run() {
			String a="AAAAA";
			System.out.println(Thread.currentThread().getName()+":"+a );
			try {
                //将数据提交给交互器，阻塞线程等待交换器返回交换的数据
				String exchange = exchanger.exchange(a);
				System.out.println("交换结束");
				System.out.println(Thread.currentThread().getName()+":"+exchange );
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	};
	
	 Runnable runnable2 = new Runnable() {
		public void run() {
			String b="BBBBB";
			System.out.println(Thread.currentThread().getName()+":"+b );
			try {
                 //将数据提交给交互器，阻塞线程等待交换器返回交换的数据
				String exchange = exchanger.exchange(b);
				System.out.println("交换结束");
				System.out.println(Thread.currentThread().getName()+":"+exchange );
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}
	};
		executorService.submit(runnable1);
		executorService.submit(runnable2);	
	}
````

**两个线程在执行交换方法时，都有可能出现阻塞，即先执行的需要等待后执行的，然后完成数据交换**

**Exchanger实际使用场景：两个线程执行计算后，需要相互获取对方数据**

​	5、ScheduledThreadPoolExecutor  调度线程池执行器

在java5没有提出ScheduledThreadPoolExecutor之前，有两种方法进行定时任务：

1、通过在run（）方法的循环底部添加sleep（），通过线程休眠，来控制任务每间隔多长时间执行一次

**该方式只能达到间隔时间执行任务，功能太过于简单**

2、使用Timer（定时器类）、TimerTask（定时器任务类）来使用java的定时器机制：通过创建Timer对象，会自动创建一个安全的线程（该线程的任务只能被该线程访问，永远处于单线程状态），它能同步执行所有的定时器任务。在任务执行时，可以指定延迟执行时间（指定执行时间）、和每次任务的间隔时间或任务执行的周期时间。

````java
	Timer timer = new  Timer();
		TimerTask timerTask = new TimerTask() {
			@Override
			public void run() {
				System.out.println("发射");
			}
		};
		timer.schedule(timerTask, 0, 1000);
		//终止Timer的线程
		//Timer.cancle()
````

**1、Timer方式只能以单线程的方式执行任务，因此在指定周期任务时，会出现任务还没执行完，下一个任务就需要执行了，因此就会发生任务阻塞，无法达到周期执行任务的效果；**
**2、并且Timer线程不会捕获异常，无法使用异常机制来处理任务；**
**3、但Timer线程有一个很大的优点，它可以基于自然时间定时启动，这是ScheduledThreadPoolExecutor做不到的**

因此除了需要指定自然时间定时启动外，我们推荐使用ScheduledThreadPoolExecutor来配置定时任务

````java
		ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
		ScheduledExecutorService service2 = Executors.newScheduledThreadPool(10);
		//固定周期执行，每多少时间执行一次任务（任务、初始延迟时间（指定执行时间）、周期时间、时间单位）
//		service.scheduleAtFixedRate(new YhTask(), 0, 5, TimeUnit.SECONDS);
````

**其提供单线程池模式和固定线程模式。根据需要统一定时执行的任务数来决定；相对于timer类，在指定定时时间时，可以选择时间单位，来增加代码的可阅读性**

23、native关键字    

native用于修饰方法，表示该方法为本地方法，实现使用的C/C++编写的动态链接库（dll）。一般用于java源代码中的底层函数调用，即java使用C/C++编写的函数，增加JAVA平台的运行速度或进行底层系统交互。

24、BlockingQueue  阻塞队列

**BlockingQueue用于生产者-消费者的多线程模型，其内部使用的同步锁，来保证入队、出队的线程安全**

**java5中concurrent包中的线程池（ExecutorService）分配任务使用的就是阻塞队列（BlockingQueue），它实现了对线程阻塞和唤醒的管理，解决生产者、消费者线程并发执行时的安全问题（当使用普通队列时，消费者速度大于生产者，会出现消费者从队列中获取不到元素，反之生产者大于消费者时，一直执行就会使内存溢出，因此在没有阻塞队列之前，我们需要手动控制生产者和消费者线程的阻塞，增加了代码的复杂性 ）：**

**当阻塞队列满了时，阻塞所有生产者线程；当阻塞队列空了时，阻塞所有消费者线程**

BlockingQueue有五个实现类：

​	1、ArrayBlockingQueue，基于数组实现的有界阻塞队列，先进先出（FIFO）

​	2、LinkedBlockingQueue，基于链表实现的阻塞队列，先进先出（FIFO），可以不指定队列大小，默认为Integer.MAX_VALUE(21亿)，因此可以默认为是一个无界阻塞队列

**前两种为常用阻塞队列，区别在于：**

LinkedBlockingQueue使用的是分离锁，生成者和消费者分别采用独立的锁来控制，保证了并发时生产者和消费者能同时操作队列中的数据，增加了高并发能力。

ArrayBlockingQueue使用的独立锁，牺牲并发性能，减少了代码的复杂性。但是其插入、删除队列元素的速度更快，不需要额外的对象。LinkedBlockingQueue则需要产生额外的Node对象，增加GC（垃圾回收）操作。

ArrayBlockingQueue还可以控制内部锁的公平性（是否采用公平锁），使等待最久的线程先获取元素执行。默认采用非公平锁。

​	3、PriorityBlockingQueue，基于优先级实现无界阻塞队列，根据构造器参数Comparator对象的优先级规则来排序。使用公平锁来控制线程同步。

​	4、DelayQueue，基于延迟时间排序的无界阻塞队列，底层是通过PriorityBlockQueue来实现。当到达延迟时间后，才能被消费者线程获取，其他时候线程处于阻塞状态。

​	5、SynchronousQueue，同步阻塞队列，不存储元素，每一次put操作必须等待一个take操作。因此生产者线程每次执行任务后，都需要等待一个消费者线程获取该元素。因此适用于生产者线程和消费者线程同步传递某些信息、事件和任务。可以控制内部锁的公平性（是否采用公平锁）。

**无界阻塞队列，会导致生产者线程不会阻塞，当消费者线程速度小于生成者时，发生内存溢出**

**java并发包提供 LinkedBlockingDeque（基于链表实现的阻塞双向队列），来实现LIFO（后进先出）的队列顺序**

25、并发锁的分类

并发锁大类分为：悲观锁和乐观锁，并且不只运用在java并发编程中（如数据库的数据锁机制）

悲观锁：指程序在并发情况下，必须进行同步处理，只有一个线程执行完后，下一个线程才能执行。如使用同步锁（Synchronized）和显式使用Lock

乐观锁：保证不使用同步锁时，并发情况下的线程安全。多线程并发时，并不会只执行一个线程，而是乐观的认为不加锁的并发操作不会发生线程安全问题（即不使用锁），并发执行所有的线程，当发生线程安全问题时，强制使该次线程操作失败，并重复再次执行。

乐观锁通过计算机底层的CAS（**Compare And Swap，比较并替换**）机制来实现，CAS机制中有3个基本操作数：V（内存地址）、A（预期值，保存旧内存地址中的值）、B（要修改的新值），只有当V中的值为A时，才执行把B的值赋值给V的操作，否则就操作失败重新执行。

**CAS中的每一步操作都具有原子性，因此保证了并发环境下的线程安全**

26、多线程原子类

java 5并发包通过java.util.concurrent.atomic库类（原子类），引入了对CAS机制的支持，实现了java中的乐观锁

支持Integer、long、Blooean和对象引用类型的CAS操作，以AtomicInteger为例：

````java
AtomicInteger atomicInteger = new AtomicInteger(10);
		AtomicInteger atomicInteger1 = new AtomicInteger(1);
		ExecutorService executorService = Executors.newCachedThreadPool();
//      定义int类型的一元运算函数（加减乘除）
		IntUnaryOperator intUnaryOperator = new IntUnaryOperator() {
			
			@Override
			public int applyAsInt(int operand) {
				return operand+9999;
			}
		};
//      定义int类型的累加器函数（只能加法）
		IntBinaryOperator intBinaryOperator = new IntBinaryOperator() {
			@Override
			public int applyAsInt(int left, int right) {
				return left+right;
			}
		};
		Runnable runnable = new Runnable() {
			@Override
			public void run() {
				//返回加法前的值
				System.out.println(atomicInteger.getAndAdd(10));
				//返回加法后的值（get在后，返回执行后的结果，在前返回执行前的结果）
				System.out.println(atomicInteger.addAndGet(20));
				//使用累加器返回结果 n=n+4
				System.out.println(atomicInteger1.accumulateAndGet(4, intBinaryOperator));
				//--n,返回一次递减值
				System.out.println(atomicInteger1.decrementAndGet());
				//++n,返回一次递增值
				System.out.println(atomicInteger1.incrementAndGet());
				//赋值一个新值 n=100,只有返回执行前的值方法
				System.out.println(atomicInteger1.getAndSet(100));
				//执行更新器，修改atomicInteger的值
				System.out.println(atomicInteger1.updateAndGet(intUnaryOperator));
			}
		};
		executorService.submit(runnable);
		}
````

**AtomicInteger的方法在执行时，默认使用CAS机制，多线程下并发执行，若不满足CAS条件，则重新执行。其中对于这些方法要保证执行前后的值不一样，否则就会发生ABA错误（执行前后的值一样），由于CAS机制无法感知到其他线程做了操作，而直接放行所有线程的执行。**

Atomic包提供了另一种解决方案：对于原子变量的更新提供一个版本值，而不是对比它本身值。这样即使原子变量更新后不变，CAS机制依然可以检测原子变量是否更新。使用AtomicStampedReference 和 AtomicMarkableReference 类来实现。

**原子类只能保证单个共享变量的更新线程安全，但当一个任务中有多个共享变量进行相交操作时，就只能使用同步来实现**

原子类主要解决**Volatile关键字配合原子操作使用不能处理变量本身运算（n++）的问题，**CAS机制实现了Volatile关键字提供的变量可视性，使多线程都能获取到当前变量的值。在此基础上，保证了对于变量非原子操作的线程安全。

**原子类使用乐观锁，非阻塞式地保证了线程安全，在一定情况下增加了并发效率；但在高并发下，由于原子变量操作频率太高，原子变量更新失败率变高，反而会导致性能损耗增高，效率下降。因此需要合理选择使用原子类还是同步锁。**

27、多线程同步方式（实现线程安全）

阻塞同步：同步锁（Synchronized）和显式使用Lock

非阻塞同步：Volatile关键字配合原子操作、原子类

28、非阻塞队列ConcurrentLinkedQueue

ConcurrentLinkedQueue：基于链表实现的无界非阻塞队列，采取FIFO（先进先出）排序。通过CAS机制来实现出队、入队时的并发线程安全。（当并发执行出队时，同一时间只有一个线程能成功出队获取元素，而其他线程获取失败，重新尝试获取）因此，在并发量适中的情况下，ConcurrentLinkedQueue一般具有较好的性能。

30、并发容器

java5的juc包提供了所有的并发容器：

**1、CopyOnWriteArrayList**：List接口的实现类，相当于线程安全的ArrayList

**2、CopyOnWriteArraySet**：Set接口的实现类，内部通过CopyOnWriteArrayList实现，相当于一个线程安全的ArrayList且不会出现重复元素，因此是有序的。通过CopyOnWriteArrayList提供的addIfAbsent()和addAllAbsent()额外方法，来添加不重复的元素。

**两个容器的线程安全都是通过Volatile和同步锁来实现，并且CopyOnWriteArraySet和CopyOnWriteArrayList大致相同，只是多了一个容器元素不重复的特性：**

CopyOnWriteArrayList实现容器数据的读写分离，它对所有的写操作都使用ReentrantLock加锁，并操作的对象是容器底层数组的一份copy：

​	1、当其他线程需要进行写的操作时，就只能等待前一个线程释放锁，之后再进行写入操作。写操作最后结束时将该copy对象替换原有对象。

​	2、当其他线程需要进行读的操作时，并发执行读操作，操作的对象是容器底层数组本身。通过Volatile修饰底层数组，来实现容器数据的可见性，配合数组读操作的原子性，使得在读操作进行时，获取当前最新数组的值

从CopyOnWriteArrayList的实现原理可知：

​	1、CopyOnWriteArrayList的add（添加）、set（更新）、remove（删除）都需要copy一份数组进行操作，然后之后进行替换。这样操作的代价非常昂贵，因此一般进行批量增加和删除。

​	2、CopyOnWriteArrayList读操作没有使用锁，因此并发环境下非常高效，但写操作效率比使用常规的同步锁进行容器操作更加慢。**因此CopyOnWriteArrayList更加适合多读少写的并发环境，当有太多的写操作时，应该使用常规的同步锁操作。（容器非常大，即使只存在少量的写操作，也会大大影响容器效率）**

CopyOnWriteArrayList的迭代：

​	1、CopyOnWriteArrayList拥有一个内部类COWIterator，提供自身iterator()方法的迭代器对象，COWIterator是ListIterator的子类。

​	2、COWIterator不允许修改容器，执行remove（）方法时会出现UnsupportedOperationException（不支持操作异常）异常

​	3、由于所有的迭代器都是当前容器的快照，因此迭代过程中无法感知当前容器的修改，无法保证数据的一致性

**3、ConcurrentSkipListSet：**相当于线程安全的TreeSet，具有有序性，底层通过ConcurrentSkipListMap实现

**4、ConcurrentSkipListMap：**相当于线程安全的TreeMap，但它底层并不是使用红黑树数据结构，而是使用的跳表SkipList

ConcurrentSkipListMap在进行插入（put）、删除（remove）方法时，使用了CAS机制实现线程安全，保证了同一时间内只有一个线程修改操作能完成，其他线程操作失败，循环重复执行直到成功；在进行读操作时，并发执行，这样大大增加了并发能力。

**跳表SkipList：**普通链表插入、添加速度很快，但查询速度很慢，因此出现了跳表这种数据结构。

它通过多层链表实现，并且每一层都是有序的。它以增加空间复杂度的代价换取了随机查询效率的提供。

![](C:\Users\OneMTime\Desktop\Typora图片\JDK8HashMap跳表结构图.jpg)

**跳表和红黑树的区别：**

插入、查找、删除以及迭代输出有序序列这几个操作，红黑树也能完成，时间复杂度和跳表是一样的。

1、但是，按照区间来查找数据这个操作，红黑树的效率没有跳表高。

2、跳表的代码更容易实现，可读性好不易出错

3、跳表更加灵活，可以通过改变索引构建策略，有效平衡执行效率和内存消耗

因此在Redis中，有序Set的实现通过跳表和散列表来实现。

**5、ConcurrentHashMap:**相当于线程安全的HashMap，相对于过时的Hashtable，它并不是对于容器数据的读写全部进行synchronized线程同步，这样高并发时性能会非常低。

**ConcurrentHashMap在java5--java7，使用的分段锁技术来细化锁粒度**：ConcurrentHashMap本质上为Segment（ConcurrentHashMap中的内部类，分段锁）数组，每一个Segment对象中都含有若干个HashEntry（保存map键值对），通过HashEntry的数据的可见性（被Volatile修饰）和加锁重读机制保证了并发环境下高效安全的读操作；针对于每一个Segment对象，使用同步锁保证每一个Segment对象进行写操作时的线程安全，因此写操作可以在不同的Segment对象中同时操作，大大减少了写操作时的线程阻塞，提高并发性能。

Segment个数默认16，可以根据ConcurrentHashMap构造函数来设置，取值的最小2的n次方作为Segment个数。Sement的个数最好与并发线程数相等，保证性能最高。

**ConcurrentHashMap在java8，通过与JAVA8的HashMap类似的数组+链表+红黑树底层数据结构，并使用CAS机制来保证ConcurrentHashMap容器写操作的线程安全。**

**在java8时，HashMap被修改，由原来数组+链表的数据结构修改为数组+链表+红黑树的数据结构。**

**数组+链表数据结构原理：**

通过数组来保存哈希值，每一个哈希值对应一条链表，用于存储对应该哈希值的所有Node<K,V>对象，该对象保存了键值对。

![](C:\Users\OneMTime\Desktop\Typora图片\HashMap数组+链表数据结构图.png)

**JAVA8，HashMap修改原因：**

当哈希值碰撞（相等）太多，则哈希值对应的链表就会边长，大大影响hashMap的查询速度，因此当链表长度超过 8时，将转为红黑树储存，通过红黑树结构进行查询优化。

**LinkedHashMap继承了HashMap，因此结构也随之修改，但还是和以前一样，多创建了一个双向链表按插入顺序保存所有键值对，保证遍历时容器数据的排序。**

**并发Map容器的key、value都不能为null，防止在并发环境下，无法判断key、value是不存在还是本身就为null；而在单线程环境下，可以通过contains方法来进行判断，而多线程下无法保证contains与get操作的原子性，而需要在外部使用同步锁，则大大减低效率，因此直接将并发Map容器设计为KV不允许为null**

**ConcurrentSkipListSet不能存入null：因为需要对值进行排序（实现跳表结构）**

31、并发锁类型

**从上面知识点可以发现，锁只是一个工具，通过对它的灵活使用，可以达到不同的效果，以下是常见各类型锁：**

**1、公平锁/非公平锁：能让线程根据等待时间，按顺序依次获取锁**

对于使用显示的Lock：ReentrantLock、ReentrantReadWriteLock都可以通过构造方法来设置公平性（默认不公平）

对于Synchronized，它是一个非公平锁，并且无法变为公平锁

**2、可重入锁/不可重入锁：可以重复、递归使用的锁**

对于ReentrantLock、ReentrantReadWriteLock和Synchronized都是可重入锁

理论上只能通过自定义Lock，来创建一个不可重入锁（因此不可能用到）

**3、读写锁：及java中的ReentrantReadWriteLock对象**

ReentrantReadWriteLock含有两个锁：readLock、writeLock

**readLock为共享锁**，可以让所有线程并发执行读操作，但不能继续获得写锁进行写操作

**writeLock为互斥锁**，只能被一个线程持有，可以继续获取读锁，进行读操作

**4、分段锁：通过减少锁得持有时间和使用锁得请求频次来降低锁的竞争程度，从而允许更高的并发性。**、

**5、偏向锁 / 轻量级锁 / 重量级锁：这是使用Synchronized同步锁时，JVM为提高锁获取与释放效率做的优化，提供锁的三种中间状态，不能被开发者控制**

偏向锁：当目前只有一个线程使用该锁时，该线程会自动获取锁，减少获取锁的代价

轻量级锁：但该同步代码被多个线程访问时，锁状态由偏向锁转变为轻量级锁；此时其他线程并不会由于无法获取锁而阻塞，而是通过自旋的方式不停尝试获取锁

重量级锁：锁处于轻量级锁状态，出现线程自旋一定次数的时候，还没有获取到锁时，转变为重量级锁；此时所有线程当获取锁失败时，直接进入阻塞状态

**这种方式优化了同步锁在低并发时，过于绝对地阻塞线程，额外增加线程等待和唤醒的过程。**

**6、自旋锁：即 使用CAS机制的无锁编程，在一定的并发环境下，既保证了线程安全，又增加了并发效率。**

缺点是：1、在锁持有时间过程时，出现大量线程循环尝试获取锁，这样大大增加了cpu的使用率

​				2、该方式无法提供公平性，即无法满足等待时间最长的线程优先获取锁

![](C:\Users\OneMTime\Desktop\Typora图片\JDK并发包API总结.png)

