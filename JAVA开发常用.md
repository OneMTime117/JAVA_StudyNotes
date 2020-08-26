- # JAVA开发常用基本类型、持有对象及其它们工具类的常用方法：


## 1、八大基本数据类型

int   整数   字节4个   2的32次方
long   长整数    8个   
short   短整数   2个    范围（2的16次方，即-2的15次方 到  2的15次方-1   -32768----- 32767）
float    单精度浮点型（小数）   4个
double   双精度浮点型 （大范围的小数）  8个
char      字符（单引号，一个字符）      Unicode编码：2个  计算机能表示的字符数0-65535
boolean   布尔型（赋值为真假（turn false）对应的整数赋值为1，0 ；非零代指1）  一个字节  
byte      字节型（-128-127整数）     1个   范围（2的8次方）

定义float时，应在赋值数后加F，系统默认小数为double类型，以声明赋值数满足其float数据类型范围
定义long时，应在赋值数后加L，系统默认整数为int类型
float范围大于long范围
float精度有效位数为7
double 精度有效位数为16

- 基本数据类型的相互转化：

1、自动转化：byte->short->int->long->float->double   char->int

在JAVA中，基本类型（除了boolean外）可以自动转换的；表述范围小的可以自动转换为表述范围大的；当不同基础类型计算时，范围小的会自动转化为范围大的，再进行运算，如：float与double运算时是自动转换为double再进行计算，结果为double类型

2、强制转化：

表述范围大的可以强制转换为表述范围小的，需要对结果进行显示的强制转化，有时会导致丢失精度或数据溢出（获得一个没有意义的数值）

int a=1;
double b =32423;
int c= (int) (a+b);

- 基本数据类型的包装类

Byte、Integer、Character、Short、Long、Float、Double、Boolean

​	**以Integer为例，基础数据和包装类的相互转化：**

1、包装类通过new和valueOf（int a）静态方法的两种方式，实现基础类型向包装类的转化

2、intValue（）：实现包装类转化为基础类型

在JDK1.5后，引入了自动拆装箱的语法，直接可以直接赋值，系统自动进行拆箱和装箱

````java
	int a = 1;
Integer b =2;//装箱
a=b;//拆箱
````

​	**包装类是对基础类型的封装，实际上内部还是操作的基础数据类型：**

​	1、除Character、Boolean外，其他包装类都继承了抽象类Number，实现了内部基础类型自动转化或强制转化为其他基础数据类型的方法：byteValue（）、intValue()......

​	2、对于Character类，则是内部自定义了一个charValue()方法，转化为char类型；然后只能手动强制转化为int（通过ASCII码进行转化）

toUpperCase（char c）、toLowerCase（char c）：静态方法，将指定字符转为大写、小写

isUpperCase（char c）、isLowerCase（char c) :静态方法，判断指定字符是否为大写、小写

​	3、对于Boolean类，有两种构造方式，参数分别为boolean、String；String参数的构造方法，Boolean内部会对参数进行判断，为“true”字符串时，则value为true

   **4、所有包装类都实现了Comparable接口，重写compareTo（）方法，以Character为例：**

compareTo （Character  anotherChar）：将两个字符比较，相同时返回0；不同时返回char-anotherChar的差值（ASCII码的差值）

​	**5、所有包装类都重写了Object类的equals（）方法、toString（）方法；用于比较和展示包装类中内部的基本数类型值**

​	**6、所有包装类都有ValueOf（String s）静态方法，用于将String类型转变为基本数据类型的包装类；所有包装类也都有parseByte（String s）静态方法，用于将String类型转变为基本数据类型**

**所有基础包装类，都是不可变的（无法被继承），即外部无法对类中的成员变量进行修改**

## 2、String类

String类中默认将字符串保存为一个char类型数组，然后进行操作，实现了CharSequence接口，重写如下常用方法：

CharSequence（字符序列）是java字符串的另一种表达形式，可以实现字符串数据类型的读写；String类只能实现字符串的读

1、length（）：获取字符串长度

2、chanrAT（int a）：获取指定下标字符

String实现了Comparable接口，实现了：

3、compareTo（String s）：两个字符串按字符顺序依次比较（ASCII码的差值），直到第一次出现不同，小为负，大为正；所有字符都相同，则返回0

4、重写Object的toString（）、equals（Object obj）、hashCode（）方法；用于比较和展示字符串

5、toUpperCase（）、toLowerCase（）：转换字符串所有字符的大小写

6、indexOf（int char）：查找指定字符第一次出现在字符串中的下标

​      indexOf（int char ，int index）：从下标为index的开始（包含该下标的字符），查找字符第一次出现在字符串的下标

​	  lastIndexOf（int char）：查找指定字符最后一次出现在字符串的下标

7、split（String s）：通过正则表达式，以s来分隔该字符串

8、trim（）：去掉字符串左右空格

9、replace（CharSequence target, CharSequence replacement）:将字符串中所有的target字符串代替为replacement字符串

​	  replaceAll (String regex, String replacement)	：支持对第一个字符串参数进行正则表达式解析，满足replace的方法同时，还可以使用正则表达式进行匹配替换

​	  replaceFirst(String regex, String replacement)：正则表达式进行匹配替换，只替换第一处

10、substring(int beginIndex,int endIndex) 截取字符串，从beginIndex开始（包含当前下标），到endIndex前结束（不包含当前下标字符）

11、contains(String s)：判断是否包含该字符串

12、equalsIgnoreCase(String s)：比较字符串，但忽略大小写

13、startsWith(String）、endsWith(String)：判断字符串是否以指定前缀开始、后缀结束

## 3、String类的辅助类：StringBuffer、StringBliud

由于String对象是不可变的：即外部无法对类中的成员变量进行修改;String 中只有三个成员变量：value，offset和count，都被private修饰，且没有set方法，因此在操作String类型时，实际上时重新new了一次

大大部分场景中，我们都需要进行大量的String操作，如果使用String，就会产生大量无用的String对象；因此就需要使用可操作的String类：StringBuffer、StringBliud

- **StringBuffer、StringBliud都可以对本身字符串进行操作，常用方法有：**

1、append方法
public StringBuffer append(boolean b)
该方法的作用是追加内容到当前StringBuffer对象的末尾，类似于字符串的连接。

2、deleteCharAt方法
public StringBuffer deleteCharAt(int index)
该方法的作用是删除指定位置的字符，然后将剩余的内容形成新的字符串。

3、insert方法
public StringBuffer insert(int offset, String s)
该方法的作用是在StringBuffer对象中插入内容，然后形成新的字符串。

4、reverse方法
public StringBuffer reverse()
该方法的作用是将StringBuffer对象中的内容反转，然后形成新的字符串。

5、setCharAt方法
public void setCharAt(int index, char ch)
该方法的作用是修改对象中索引值为index位置的字符为新的字符ch。

6、trimToSize方法
public void trimToSize()
该方法的作用是将StringBuffer对象的中存储空间缩小到和字符串长度一样的长度，减少空间的浪费。

7、toString方法

转换为不变字符串（string类型）:变量调用 StringBuff的toString()方法

8、构造方法中可有参数，定义字符缓存区的字节数
StringBuffer s0=new StringBuffer();默认分配了长16字节的字符缓冲区
StringBuffer s1=new StringBuffer(512);分配了512字节的字符缓冲区

- **StringBuffer、StringBliud两者的区别：**

StringBuilder类，用于更效率的对字符串进行操作，线程不安全；

StringBuffuer类，其线程是安全的，因此效率比StringBuilder低一点

- **java默认重载了配合String使用时的操作符“+”和“+=”，执行了StringBuilder的append（）方法，然后转换为String对象**

## 4、JDK常用工具类及其方法：

### 1、日期类Date

1、getTime（）：获取当前时间的毫秒数，从1970年开始

2、setTime（Long time）：设置当前date对象的时间毫秒数

3、toString（）：以dow（星期） mon（月份） dd（日） hh:mm:ss zzz（时区） yyyy（年） 格式来展示当前时间

4、after(Date date) ：判断当前时间是否在date之后

​	  before(Date date)：  判断当前时间是否在date之前

​	  equals(Object obj)：  判断当前时间是否于date相等，精确到毫秒

### 2、日历类Calendar

相对于日期类，Calendar解决了date获取年份需要+1970的问题，并且在对于时间节点的加减上更加灵活，能够简单解决日常开发中获取复杂条件时间的问题，如该月的第一个星期。。。

**1、Calendar实例化使用getInstance（）静态方法获取：**

有多个重载方法，需要传入TimeZone (时区)、 Locale（国家）；无参方法使用系统默认TimeZone(Asia/Shanghai)、Locale（zh_CN）

不同国家和时区，可能会使用不同的日历系统

**时区概念：**

全球按照经线0°开始将全球分为24个时区，每15°作为对于时区的中央经线，因此每个时区相隔1个小时

不同国家所占时区的多少也有不同，因此通过该国家的城市名来划分该国家的所有时区，中国五个时区名为：

Asia/Chongging、Asia/ChungKing、Asia/Harbin、Asia/Kashgar、Asia/Urumqi；但是由于中国统一使用北京时间，因此它们都对应为GMT +08:00

时区使用的标准规则：

1、GMT（格林威治标准时间），通过天文学知识，来定义不同时区的标准时间

2、UTC（世界协调时间），基本和GMT一样，基于原子钟，更加精准，误差在0.9s以内

3、DST（夏日节约时间），在正常时间上，把时间提前一个小时，用于适应夏天日出早，而来控制减少灯光、照明的开支

4、CST：四个时区的缩写，一般不推荐使用，因为java默认使用 中国标准时间，JS使用 美国标准时间

1. Central Standard Time (USA) UT-6:00   美国标准时间
2. Central Standard Time (Australia) UT+9:30  澳大利亚标准时间
3. China Standard Time UT+8:00     中国标准时间
4. Cuba Standard Time UT-4:00     古巴标准时间

**2、Calendar常用方法：**

Calendar类提供多个静态成员变量，用于描述日历参数：

年月日、时分秒、毫秒、当月第几天、当周第几天、当月第几个星期、当年第几天、当年第几周

这些参数很好的描述了平常开发中需要的时间

get(int field)，获取当前日历对象的日历参数值

set(int field)，设置当前日历对象的日历参数值

setTime（Date date），将date对象转化为日历对象

getTime（），将日历对象转化为date对象

add(int field,int amount)，对于日历参数进行加减

getTimeInMillies()，获得毫秒数

### 3、时间格式化类DateFormat、SimpleDateFormat

**两个格式化类都是线程不安全的**

1、dateFormat是一个抽象类，提供实例化自身的方法:

getDateInstance（int style, Locale aLocale）日期格式化 2019-11-11

getDateTimeInstance（int style, Locale aLocale）日期+时间格式化 2019-11-11 12：22：32

它们有两个参数，int style（格式化风格）, Locale aLocale （语言环境）；从而确定时间的表达方式

2、 simpleDateFormat("yyyy-MM-dd HH:mm:ss")：构造方法，创建时间格式化对象，并指定格式化模板

有多个构造方法，还可以指定语言环境，或使用默认解析模板和语言环境（Locale.CHINA）；相对于DateFormat使用模板更加方便、灵活

模板含义：

| 字母 | 含义                                                         | 示例                                                         |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| y    | 年份。一般用 yy 表示两位年份，yyyy 表示 4 位年份             | 使用 yy 表示的年扮，如 11； 使用 yyyy 表示的年份，如 2011    |
| M    | 月份。一般用 MM 表示月份，如果使用 MMM，则会 根据语言环境显示不同语言的月份 | 使用 MM 表示的月份，如 05； 使用 MMM 表示月份，在 Locale.CHINA 语言环境下，如“十月”；在 Locale.US 语言环境下，如 Oct |
| d    | 月份中的天数。一般用 dd 表示天数                             | 使用 dd 表示的天数，如 10                                    |
| D    | 年份中的天数。表示当天是当年的第几天， 用 D 表示             | 使用 D 表示的年份中的天数，如 295                            |
| E    | 星期几。用 E 表示，会根据语言环境的不同， 显示不 同语言的星期几 | 使用 E 表示星期几，在 Locale.CHINA 语 言环境下，如“星期四”；在 Locale.US 语 言环境下，如 Thu |
| H    | 一天中的小时数（0~23)。一般用 HH 表示小时数                  | 使用 HH 表示的小时数，如 18                                  |
| h    | 一天中的小时数（1~12)。一般使用 hh 表示小时数                | 使用 hh 表示的小时数，如 10 (注意 10 有 可能是 10 点，也可能是 22 点） |
| m    | 分钟数。一般使用 mm 表示分钟数                               | 使用 mm 表示的分钟数，如 29                                  |
| s    | 秒数。一般使用 ss 表示秒数                                   | 使用 ss 表示的秒数，如 38                                    |
| S    | 毫秒数。一般使用 SSS 表示毫秒数                              | 使用 SSS 表示的毫秒数，如 156                                |

**只使用单个字母时，代表第二位数为0时，不需要展示：04 -> 4**

3、两者的常用方法：

format(Date date)，将时间按照格式化模板转化为字符串

parse(String time),将满足格式化模板的字符串，转化为date对象

### 4、数字格式化类NumberFormat、DecimalFormat

**两个格式化类都是线程不安全的**

1、NumberFormat，是一个抽象类，有四个方法来获得实例化对象

1. 使用getInstance或getNumberInstance获取正常的数字格式。
2. 使用getIntegerInstance得到的整数格式。
3. 使用getCurrencyInstance来获取货币数字格式。
4. 使用getPercentInstance获取显示百分比的格式。

它们有一个Local参数，来指定对于的语言环境

之后通过以下方法，来调整格式化对象的默认参数：

format.setParseIntegerOnly(true)；//设置字符串解析成数字时，是否应该仅作为整数进行解析，此方法只影响解析，与格式化无关

format.setMinimumFractionDigits(2);//设置数值的【小数部分】允许的最小位数

format.setMaximumFractionDigits(3);//设置数值的【小数部分】允许的最大位数

format.setMinimumIntegerDigits(1);//设置数值的【整数部分】允许的最小位数

format.setMaximumIntegerDigits(5);//设置数值的【整数部分】允许的最大位数

**这四个位数的设置，会直接将数字的整数进行截取，小数部分进行舍入；最小位数不足，则补o**

format.setGroupingUsed(false)；//关闭分组，NumberFormat默认使用千位符分隔

format.setRoundingMode(RoundingMode.XXX);//设置舍入模式，一般常用四舍五入RoundingMode.HALF_UP

，默认为HALF_EVEN：

5以上不管是奇数还是偶数，5都舍入，5以下不管是奇数还是偶数，5都舍去；当是5时，若前一位是奇数，5就舍入，若前一位是偶数，5就舍去      

**format.parse("12213.1")：解析后会返回一个Number对象，然后通过XXXvalue来转化为对应的基本数据类型**

2、DecimalFormat是NumberFormat的实现类，使用模板来进行数字的格式化

| **No.** | **标记** | **位置**   | **描述**                                                     |
| ------- | -------- | ---------- | ------------------------------------------------------------ |
| 1       | 0        | 数字       | 代表阿拉伯数字，每一个0表示一位阿拉伯数字，如果该位不存在则显示0 |
| 2       | #        | 数字       | 代表阿拉伯数字，每一个#表示一位阿拉伯数字，如果该位不存在则不显示 |
| 3       | .        | 数字       | 小数点分隔符或货币的小数分隔符                               |
| 4       | -        | 数字       | 代表负号                                                     |
| 5       | ,        | 数字       | 分组分隔符                                                   |
| 6       | E        | 数字       | 分隔科学计数法中的尾数和指数                                 |
| 7       | ;        | 子模式边界 | 分隔正数和负数子模式                                         |
| 8       | %        | 前缀或后缀 | 数字乘以100并显示为百分数                                    |
| 9       | \u2030   | 前缀或后缀 | 乘以1000并显示为千分数                                       |
| 10      | ¤\u00A4  | 前缀或后缀 | 货币记号，由货币号替换。如果两个同时出现，则用国际货币符号替换。如果出现在某个模式中，则使用货币小数分隔符，而不使用小数分隔符。 |
| 11      | ,        | 前缀或后缀 | 用于在前缀或或后缀中为特殊字符加引号，例如 "'#'#" 将 123 格式化为 "#123"。要创建单引号本身，请连续使用两个单引号："# o''clock"。 |

DecimalFormat decimalFormat = new DecimalFormat("##.000%");//获得百分数格式化

### 5、数学类Math

math类提供一系列基本数据运算和几何函数的方法，所有成员变量和方法都是静态的，常用有：

1、静态常量：pi  圆周率的double值

2、pow (double a, double b)   a的b次方

3、sqrt（double a） a的平方根

4、abs（int a）a的绝对值，可以是long、float 和 double 类型的参数

5、ceil（double a）返回大于等于a的最小整数的double类型值

6、doouble floor（double a）返回小于等于a的最大整数的double类型值

7、max（int a,int b）、min（int a,int b）获得两者中较大、较小的

8、round（float a）返回四舍五入的整数

**在负数进行四舍五入式，先对其绝对值进行四舍五入，然后在将负数加上**

Math.random() 返回一个[0.0，1.0）之间的double值（内部是使用了Random类）

### 6、随机数类Random

Random有两个构造方法，默认构造器或传入一个Long类型的随机种子参数

提供除char类型外的七中基本数据类型的随机数，Random的随机数并不是完全随机的（伪随机数），整个随机数序列都和随机种子参数有关，因此两个相同随机种子参数的Random类，产生的随机数序列是相同的

````java
		Random random = new Random();
		double nextDouble = random.nextDouble();//0.0-1.0,不包含1.0
		int nextInt = random.nextInt(1000);//0到1000，不包含1000
````

使用默认构造方法是，也生成了一个随机种子参数：seedUniquifier() ^ System.nanoTime()，进行的大量的运算，理论上不可能有相同数，因此一般使用默认构造方法就可以保证完全随机。只有特殊需求时，才指定随机种子参数

### 7、系统类System

System有三个成员变量，in（标准输入流，键盘输入）、out（标准输出流）、err（标准错误输出流）

常用方法有：

exit（int status）:退出程序，0代表正常退出、非0代表异常退出

currentTimeMillis（）：返回当前时间毫秒数

nanoTime() ：返回一个随机时间的纳米级时间数

### 8、Object类

Object是java中所有类的超类，即他们类层次结构中的根类；所有类都继承了它

常用方法有：

hashCode（）：获取当前对象的哈希值（一般情况下，不同对象的哈希值是不同的）

getClass（）：获取当前对象的类的Class对象（用于进行反射操作等）

toString（）：将当前对象打印为字符串（默认是打印 该类的权限名+@+哈希值）；大多时候都需要重写

equals（Object obj）:比较两个对象的引用是否相同（两个变量是否指向同一个对象内存地址），和==的作用一样；大多时候都需要重写

wait（）用于在多线程控制中，释放当前对象的使用权，并一直处于阻塞状态，直到被唤醒

notify（）、notifyAll（）：用于在多线程控制中，唤醒执行了改当前对象wait（）方法的线程

finalize（）：用于java垃圾回收机制，告诉垃圾回收器，该对象可以被回收（但是并不能使垃圾回收器一定去销毁该对象）

clone（）：java对象浅拷贝，它会拷贝当前对象，生成一个新的对象，并有一个新的引用，因此两者表面上是不相关的；但是在对象中的引用类型属性变量保存的引用却是指向同一个对象的，**所以在操作该对象中的引用类型属性时，会影响拷贝对象中的属性（由于String类和包装类为final，因此在拷贝后，在修改拷贝对象的String属性时，不会修改引用指向的内存空间的值，而是创建一个新的对象进行引用，因此两者不会相互影响）**

**浅拷贝的使用：**

1、由于clone（）在Object类中是protected的，因此只能在子类内部中使用，无法直接在外面通过类的实例调用；因此需要使用浅拷贝的类，需要使用一个方法提供对内部this.clone的访问

2、clone方法会抛出一个CloneNotSupportedException异常；只有当使用该方法的类实现Cloneable接口时，才不会抛出异常，即就是允许该类执行浅拷贝

**一般不使用浅拷贝，会带来对象使用时不必要的风险，可以通过代码来实现深拷贝，即对象和拷贝对象完全不相关：**

1、使用拷贝工厂，利用反射来实现（更容易使用）

````java

````

2、添加一个类的构造方法，参数为该类的实例对象，然后通过该构造方法创建新对象时，保证新对象中的所有属性都是新new的，值和参数对象中的一致（需要修改类的本身结构）

3、将对象序列化，然后在反序列化生成一个新对象，这种方式效率很慢，并且所有有关的应用类型都需要实现Serializable接口，保证能够序列化

## 5、JDk常用集合方法

### 1、Collection接口常用方法：

add(E e) ，添加元素
clear() ，暴力清除集合中所有元素
contains(Object o)， 返回值类型：boolean。判断集合是否包含某个元素
isEmpty() ，返回值类型：boolean。如果此集合不包含元素，则返回true。即用来判断容器是否被赋值过
iterator() 迭代器。返回值类型：Iterator
size() 返回值类型：int。返回集合中的元素数

### 2、List接口额外常用方法：

get（int index）：获取指定索引位置元素
set（int index，Object obj）：修改、删除、添加指定索引位置元素
listIterator（）：list增强版迭代器，可以调用set（）、add（）方法，在当前遍历位置替换、插入元素

### 3、Set接口和Collection接口方法完全一样

### 4、Stack是一个元素先进后出的容器，继承了LIST接口，常用额外方法有：

s.push("a");  入栈
s.peek（）; 查看栈顶
s.pop(); 出栈

### 5、Map常用方法：

put（K key，V value）：添加键值对

containsKey（K key）：map中是否存在该key

containsValue（V value）：map中是否存在该value

get（K key）：获取map中key对应value

values（）：返回所有value对象的Collection集合

KeySet（）：返回所有key对象的set集合

### 6、foreach和itreator的选择，操作Collection（集合）：

它们都可以对容器中获取得到的对象进行操作（但不能对变量重新赋值，否则操作的对象和容器中的对象此时就不为同一个内存空间），从而改变容器中保存的值（操作的是相同内存空间的值）

foreach需要知道容器具体类型，且不能使用remove、add方法，但是代码整洁、易懂

itreator不用知道容器具体类型，且速度快，且可以遍历删除remove当前元素

### 7、map遍历：

Map在遍历时，也可以对遍历的对象进行操作（但不能对变量重新赋值），从而改变map中的值；

java8下，推荐使用forEach（）方法执行

hashMap.forEach((k,v)-> {System.out.println(k+""+v);});
	}

**forEach（）本质上是匿名函数（匿名内部类），它闭包形成当前线程环境（获取外部类的所有变量）后，会使用另一个线程执行，此时局部变量是不能保证没有被销毁的，为了防止该问题的产生，jvm对于final修饰的变量，不会直接在内部类中操作，而是copy一份。因此对于内部类使用外部类局部变量时，一定要添加final，才能通过编译**

java8以下版本，推荐entrySet forach遍历

````java
Set<Entry<K, V>> entrySet = hashMap.entrySet();   
for (Entry<K, V> entry : entrySet) {
	System.out.println(entry.getkey+""+entry.getValue);
}
````

### 8、java操作数组的工具类Arrays：

1、asList（a） 将数组转化为list，不推荐使用，有很多风险
2、Arrays.equals(a1,a2)   比较两个基本类型数组是否相同（大小、顺序）
3、Arrays.deepEquals(a1,a2) 比较两个包装类数组是否相同
3、Arrays.sort(a,string)    使数组元素按照某种方式排序
4、Arrays.binarySearch(a,value) 通过二分法来寻找指定值在数组中的index，但是要求数组元素是已经排序的（由于内部使用的二分法，因此一定是排好序的；不存在时，返回-1）
5、Arrays.fill(a,value)  将一个值插入到数组所有位置,当然也可以设定起止点来插入一段相同值（一般用于快速创建测试数据）

6、Arrays.toString(a) 打印数组
7、hashCode（）相同大小、顺序的实例hashCode相同（参数分为8种基本数据类型和Object类型）

### 9、java操作集合的工具类Collections：

1、sort（list，Comparator）将集合元素以某种方式排序
2、binarySearch（list，T）通过二分法来寻找指定值在集合中的index，但是要求集合元素是已经排序的（由于内部使用的二分法，因此一定是排好序的）
3、reverse(list) 反转集合中的顺序
4、swap（list，int，int）交换指定索引数据的位置（即交换索引值）
5、fill（list，object）使用object替换list中所有的值

6、Map<String, Integer> singletonMap = Collections.singletonMap("id",1)  ，创建一个只有一个元素的map，且不能改变，减少map对象的内存占用

7、reverse(list)， 反转排序

**sort方法默认使用持有对象中，对应元素类型实现Comparable接口，而实现的compareTo（）方法（使用自然排序方法，降序，自然排序时，需要持有对象中不能存在null）；也可以直接传入对应元素泛型的Comparable接口的实现类，并且重写compareTo（）方法**

### 10、java操作对象的工具类Objects：

1、equals（object a，object b）:比较两个对象是否相等；都为null时为true；其中一个为null时为false（可以有效避免使用a.equals(b)出现的空指针异常）

2、deepequals（object a，object b）：深度比较两个对象是否相等（如果为数组对象，则调用Arrays.deepequals方法，进行包装类数组的比较）；都为null时为true；其中一个为null时为false

3、nonNull（Object a）:检测对象是否不为null，不为null返回true

4、isNull（Object a）：检测对象是否为null，为null返回true

5、toString（Object a）：调用对象的toString方法，为null则返回null字符串

6、toString（Object a，String nullDefault）：调用对象的toString方法，为null则返回nullDefault默认字符；该方法就是对三元表达式的封装简化，如在获取数据库查询字符字段数据，对为nullString对象转换为一个默认值的String

```java
String a = res.getString !=null：res.getString:"未知"；
String a = Object.toString(res.getString(),"未知")；
```

### 11、java比较器工具类Comparator：

1、Comparator类提供了多个comparing()方法，以Function（函数式接口）具体实现类为对象，从而实现提取某个类型对象中的一个基本数据类型进行compareTo方法比较（8种基本数据类和String都提供了该方法）的自定义Comparator；**然后可以作用Arrays/Collections的sort方法参数，进行数组、集合容器中的对象排序、或者treeSet、treeMap的构造方法参数，实现它们的排序**

```java
Comparator<Account> comparing = Comparator.comparing(new Function<Account,String>(){
    public String apply(Account a){
        return t.getAccount();
    }
});

//可以使用lambda表达式的方法引用进行简化
Comparator<Account> comparing2 = Comparator.comparing(Account::getAccount);
```

2、直接实例化Comparator接口，实现compareTo方法，来创建指定对象的比较器

```java
Comparator<Account> comparator = new Comparator<Account>() {
	@Override
	public int compare(Account a1, Account a2) {
		return a1.getAccount().compareTo(a2.getAccount());
	}
};
```

## 6、Apache commons库常用方法：

Apache commons，Java开发通用库，进一步扩展JDK的功能，封装了符合时下开发需求的工具或处理方法，常用几个包如下：

### 1、commons-lang3

**基础工具包，封装常用基础操作方法，它由多个工具类组成，常用有：ArchUtils、ArrayUtils、CharUtils、ClassPathUtils、CalssUtils、EnumUtils、RandomStringUtils、RandomUtils、RegExUtils、SystemUtils、StringUtils**

#### ArrayUtils

1、获得一个空数组（长度为0的数据，可以为八种基础类型和所有包装类），可以用于表示没有数据的数组来代替null，防止调用length时，出现NPE

String[] emptyStringArray = ArrayUtils.EMPTY_STRING_ARRAY;

2、toString，打印数组，和Arrays.toString的表现形式有点区别

3、hashCode，相同大小、顺序的实例hashCode相同（用于比较数组是否相同），相对于Arrays.hashcode使用更加简单，不需要考虑数组中的数据类型

4、nullToEmpty，如果当前数组为null，则返回一个长度为0的字符串；否则返回原数组

5、toObject/toPrimitive，实现基本数据类型数组和包装类数组的转化

6、isEmpty/isNotEmpty 判断数组是否为空（变量为null时，也为空，不会报NPE），底层使用的是getLength，对于null是安全的（即先判断是否为null，null默认为0）

7、getLenth/isSameLength 获得当前数组长度/比较两个数组长度；它们对于null是安全的

#### BooleanUtils

1、and（boolean array），逻辑与，可以判断数组中多个boolean类型是否都为ture

2、or（boolean array）,逻辑或，判断数组中多个boolean类型是否全为false

3、xor（boolean array），异或，判断数组中多个boolean类型是否全为ture或false

#### StringUtils

1、isEmpty ,判断字符串是否为null或者“”

2、isAnyEmpty,判断所有参数中，是否有为null或者“”

3、isNoneEmpty，判断所有参数没有一个为null或者“”

4、isBlank，判断子字符串是否为null、“”或者全为空格（**isAnyBlank、isNoneBlank和前面类似**）

5、deleteWhitespace,删除字符串中的所有空格，对于null安全

6、trim，删除字符串两端的空格，和String.trim()类似，但对于null安全

7、isAlpha，判断字符串全为字母，对于null安全，返回false

StringUtils提供了String类操作的所有方法，并且保证了String为null的安全性，除此之外还提供了如下功能方法：

提供从左、右和中间截取字符串：left、right、mid

删除指定开头字符串的开头：removeStart

对于字符串进行补全，指定补全的字符：leftPad、rightPad

指定字符串最小长度，对字符串不足的进行空格补全，且内容在中间（长度超出的，不进行操作）：center

对于字符串大小写进行取反：swapCase

字符串反转  ：reverse

对于超过指定长度的字符串，进行省略处理：abbreviate，超过的字符串，自动删除多余部分然后添加...,保证此时长度和指定长度一致；但是指定长度一定要大于3

#### ClassUtils

#### RandomStringUtils

RandomStringUtils.random（），通过该方法来指定随机字符串生成规则：

字符串长度、只许使用的字符集合、是否只使用数值、是否只使用字母、只许使用的字符集合作用范围。。。

#### RandomUtils

nextXXX（），获取对应基本数据类型的随机值，并且可以指定范围（底层就是通过Romdom类来实现，只是封装了new Romdom的过程，并且提供了范围需要的正确数值提示）

#### RegExUtils

封装了正则表达式，以正则表达式匹配，来操作String的一些分隔、替换、查找的方法

#### SystemUtils

提供许多常量，来获取当前系统、项目中的一些常用参数，如：

系统类型、系统编码、JDK版本、系统版本、系统语言、系统用户名。。。

#### ArchUtils

获取当前java运行环境信息

系统处理器架构、处理器类型、CPU总线（32\64)。。。

### 2、commons-io

针对于IO流功能的工具类库，包括6个部分：

工具类、输入、输出、过滤器、比较器、文件监听器

1、工具类：IOUtils、FileUtils

#### IOUtils常用功能有：

1、复制文件，将输入流中的数据读取写入到输出流

copy（），包含4种重载方法：1、字节输入流和字节输出流  2、字节输入流、字符输出流和  字节输入流转字符输入流的字符编码 3、字符输入流、字节输出流和  字节输出流转字符输出流的字节编码 4、字符输入流和字符输出流

copyLarge（），和copy（）基本一样，但能够指定缓存数组的大小，一般文件大于2G，则根据实际情况指定缓存数组大小，增加读写速度

实现方式：即默认使用4*1024kb数组，来进行缓存读写（不是一个个字节读写，而是当数组读满后，才进行写入；但又和缓存流不同，它不能减少IO流读写次数，但能减少IO流切换次数）

2、将String转化为输入流

toInputStream（），包含两种重载方法，1、String 2、String和String转化为字节输入流的编码格式

实现方式：将String以指定编码格式转化为字节数组，然后使用 ByteArrayInputStream构造方法，返回字节输入流

3、输入流转为String

readLines（）  将输入流中的数据，一条一条读出，放入字符串数组，包含三种重载方法，1、字节输入流 2、字节输入流和字节输入流转字符输入流的编码格式 3、字符输入流

实现方式：将字符输入流进行readLine读取，每读取一条，就放入字符串数组，直到读完

4、从输入流中获取数据（需要自己指定字节数组）

read（）,字节输入流对应字节数组，字符输入流对于字符数组,包含四种重载方法，1、字节输入流，字节数组 2、字节输入流 ，字节数组，从数据第几个字节开始读，从数据第几个字节结束   3、字符输入流，字符数组 4、字符输入流、字符数组、从数据第几个字符开始读，从数据第几个字符结束

readFully，和read类似，但是必须保证能读取的数据>=指定读取数据的结束长度（没有指定，默认为数组长度）

实现方式：根据参数类型，执行两种输出流的read方法，并指定从第几个数据开始放入参数数组，到指定数据结束read（）

5、将输入流转化为字节数组

toByteArray（）   ，有多个重载方法，输入流可以为字节输入流和字符输入流（字符可以转化为字节存储），并且可以指定返回字节数组的长度；同时还可以通过url来获取其输入流，并转化为字节数组

实现方式：创建一个字节数组输出流，进行复制COPY，然后将执行字节数组输出流的toByteArray()，获得对应字节数组

6、将数据写入输出流

write（），有多个重载方法，可以指定数据类型：字符串、字节数组、字符数组  ；输出流：字节输出流、字符输出流； 编码方式：数据的编码格式

writelines，用于将多条字符串按指定分隔符写入文件，需要参数：字符串集合、字符串分隔符、输出流、字节输出流的编码格式（字符输出流不需要该参数）

7、关闭流

url流：close(urlConn) ，它和urlConn本身调用 disConnction方法没有什么区别

io流：closeQuietly(Stream)  它可以优雅的关闭流（忽略关闭流时，需要处理的异常、是否为null）；当需要手动对于该流的关闭进行异常捕获时，则自定义方法，来进行异常处理（一般不需要）

**对于操作数据库的流资源，一般直接交给数据库连接池处理，否则就需要直接进行方法封装**

**由于缓冲流为包装流，直接可以通过其构造方法来获取，因此没有必要封装方法；并且IOUtils所有对于输入流、输出流的读写封装的方法，都适用于对应字节或字符的包装流**

#### FileUtils常用功能有：

正常使用JDK中的File对象时，注意事项：

- file.createNewFile(),在创建目录文件时，多级目录不存在,则无法直接创建，必须先创建父级目录

- file..mkedirs/mkedir()，在创建目录时，mkedirs (创建多级目录，父路径都可以不存在) mkedir（创建单级目录，即上多级的父路径要存在）；并且mkedir在windows磁盘根目录下创建目录时，也会报错，路径无法找到；**因此一般直接使用mkedirs（）方法来创建目录**

- 当使用fileInputStream来读取文件数据时，需要保证该文件存在

- 当使用fileOutputStream将数据输出到文件时，不需要保证文件存在，但需要保证父目录存在

1、创建该file的输入流fileInputStream：  openInputStream(File file)

实现方式：对于File对象进行了进一步的判断：是否可读、是否为目录、是否存在，进行了异常处理

2、创建该file的输出流fileOutputStream： openOutStream（File file，boolean append）是否追加（默认为false）

实现方式：对于File对象进行了进一步的判断：是否可读、是否为目录，进行了异常处理；父目录是否存在（不存在则创建）

3、将字节数转化为易阅读的特定文件大小格式（KB、MB、GB），四舍五入取值：  byteCountToDisplaySize(long size)

通过和file.length()或者FileInputStream.available()搭配使用，计算出文件大小

4、获得文件字节数     sizeOf（File file）

实现方式：对于File对象进行了进一步的判断：是否可读、是否为目录（为目录则返回sizeOfDirectory(file)，该目录下所有文件的总字节数）

5、计算文件夹中所有文件的总字节数 sizeOfDirectory（File file）

实现方式：对于file目录进行循环遍历，累计所有文件的sizeOf

6、对于指定file文件进行一次空操作 touch（File file）

实现方法：当文件不存在时，创建该文件及其父目录；存在则修改文件的最新操作时间

7、读取文件（内部使用openInputStream，对File对象操作进行了异常处理）

```java
byte[] readFileToByteArray( File file) //将文件内容转化为字节数组
String readFileToString( File file,String encoding) //将文件内容转化为String字符串,并指定编码格式
List<String> readLines( File file,  Charset encoding)//将文件内容一行行读取为String字符串,并指定编码格式
```

8、将数据写入文件（对于数据都进行了null的判断）

````java
//将字节数组写入文件，可以指定是否追加
void writeByteArrayToFile(final File file, final byte[] data)
void writeByteArrayToFile(final File file, final byte[] data, final boolean append)

//将字符串写入文件，并指定编码格式，可以指定是否追加
void writeStringToFile(final File file, final String data, final String encoding)
void writeStringToFile(final File file, final String data, final Charset encoding, final boolean append)
//内部使用了writeStringToFile，只是多了一个toString处理
void write(File file, CharSequence data, Charset encoding, boolean append)    
    
//将集合数据（一般为String集合）写入文件，指定追加方式、编码格式（内部使用了缓冲流）
void writeLines(final File file, final Collection<?> lines)
void writeLines(final File file, final Collection<?> lines, final boolean append)
void writeLines(final File file, final String encoding, final Collection<?> lines,final String lineEnding)
````

9、复制文件

`````java
void copyFile(final File srcFile, final File destFile) //复制文件到另外一个文件
void long copyFile(final File input, final OutputStream output) //复制文件到输出流
void copyFileToDirectory( file1 , file2)  //复制文件到一个指定的目录   
    
//把输入流里面的内容复制到指定文件
void copyInputStreamToFile( InputStream source, File destination)
    
//把URL 里面内容复制到文件。可以下载文件。
//参数1：URL资源 ； 参数2：目标文件
void copyURLToFile(final URL source, final File destination)

//把URL 里面内容复制到文件。可以下载文件。
//参数1：URL资源 ； 参数2：目标文件；参数3：http连接超时时间 ； 参数4：读取超时时间
void copyURLToFile(final URL source, final File destination,
                                     final int connectionTimeout, final int readTimeout)
`````

10、复制文件夹

````java
复制文件夹（文件夹里面的文件内容也会复制），file1和file2平级。
//参数1：文件夹； 参数2：文件夹
void copyDirectory( file1 , file2 );  

//复制文件夹到另一个文件夹。 file1是file2的子文件夹.
//参数1：文件夹； 参数2：文件夹
void copyDirectoryToDirectory( file1 , file2 );

//复制文件夹，带有文件过滤功能
void copyDirectory(File srcDir, File destDir, FileFilter filter)
````

11、移动文件

````java
/文件夹移动，文件夹在内的所有文件都将移动 
void moveDirectory(final File srcDir, final File destDir)

//文件夹移动到另外一个文件内部。boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveDirectoryToDirectory(final File src, final File destDir, final boolean createDestDir)

//移动文件
void moveFile(final File srcFile, final File destFile)

//把文件移动到另外一个文件内部。boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveFileToDirectory(final File srcFile, final File destDir, final boolean createDestDir)

//移动文件或者目录到指定的文件夹内。
//boolean createDestDir：如果destDir文件夹不存在，是否要创建一个
void moveToDirectory(final File src, final File destDir, final boolean createDestDir)
````

12、清空和删除文件夹（进行了一些异常处理）

````java
//删除一个文件夹，包括文件夹和文件夹里面所有的文件
void deleteDirectory(final File directory)

//清空一个文件夹里面的所有的内容
void cleanDirectory(final File directory)

//删除一个文件，会抛出异常
//如果file是文件夹，就删除文件夹及文件夹里面所有的内容。如果file是文件，就删除。
//如果某个文件/文件夹由于某些原因无法被删除，会抛出异常
void forceDelete(final File file)  

//删除一个文件，没有任何异常抛出
//如果file是文件夹，就删除文件夹及文件夹里面所有的内容。如果file是文件，就删除。
//如果某个文件/文件夹由于某些原因无法被删除，不会抛出任何异常
boolean deleteQuietly(final File file) 
````

13、创建目录  FileUtils.forceMkdir(file)   相对于file.mkdirs()，它对于file进行了是否为目录的异常处理

### 3、commons-collections

CollectionUtils工具类：

1、isEmpty（)  判断集合是否为空、null

2、addAll（List list ，String[ ] s）将数组s数据添加到list集合中，需要保证两者泛型相同

MapUtils工具类：

1、isEmpty（)  判断Map是否为空、null

**其他类、方法只是对JDK集合对象的优化，一般应用不需要（但可以优化我们不合理代码，带来的性能损失）**

### 4、commons-beanUtils

用于操作JAVA bean的工具包，常用工具有：

1、MethodUtils，通过反射操作对象的普通方法

````java
Test test = new Test();
MethodUtils.invokeMethod(test, "getString", null);//对象实例、方法名、参数列表数组（无参方法为null）
````

2、ConstrucotrUtils，通过反射来操作对象的构造方法

```java
Test test1 = ConstructorUtils.invokeConstructor(Test.class,null);//类的class对象、构造方法参数列表数组（无参为null）
test1.getString();
```

3、PropertyUtils，通过反射来操作对象的属性（成员变量）

````java
Test test = new Test();
Test test2 = new Test();
PropertyUtils.setProperty(test, "name", "yh");//修改对象对应属性名的值（需要属性有set方法）
PropertyUtils.copyProperties(test2, test);//将test属性复制给test2（需要属性有get、set方法）
PropertyUtils.describe(test2);//描述test2对象
````

4、copyProperties，对javaBean属性进行浅Copy

```java
User userOrigin = new User("YH","123");
User userCopy =new User();
BeanUtils.copyProperties(userCopy, userOrigin);
```

### 5、commons-codec

commons-codec是Apache开源组织提供的用于摘要运算、编码解码的包。常见的编码解码工具Base64、MD5、Hex、SHA1、DES等。

加密和编码的区别：

加密是把明文转变为一种不可破解的密文，提高识别识别难度

编码是换一种体现形式，便于传输并提高可读性

- Base64编码

  1、能做什么：在img标签中嵌入图片(需要添加src引入base64数据前缀，指明是使用了base64编码的图片数据)、将二进制数据在url中进行传输（需要进行变种处理，url会自动对标准Base64中的‘=’、‘/’进行编码）

  2、不是一种加密算法，是一种编码格式

  3、Base64，就是使用64个可打印字符来表示二进制数据的方法：

  ​	全世界语言最大的单字符为3个字节，因此一个字节最大为3*8位二进制；Base64 换为4 * 6位二进制进行表示，然后每6位转化位一个字符，最小值为0，最大值为63刚好64个值，从而对应Base64的编码表字符；64个字符组成为：26个字母大小写（52）+0-9数字（10）+ ‘=’、‘/’（2）=64

  中文字符进行Base64编码过程：

  1、根据中文字符根据编码格式进行解码，然后转为2进制数据

  2、对二进制进行分组，每6个一组，最后不足时在后面补0

  3、将每6个一组的二进制转化为十进制

  4、将每个十进制根据Base64编码表进行编码转化

  解码过程：

  1、将每个字符通过Base64编码表进行解码，获得多个十进制

  2、将每个十进制转化为二进制

  3、对二进制进行每8个一组的分组，将删除多出来的0

  4、然后对于每组二进制进行根据字符编码格式编码，转化为对应字符串

  ````java
  //JDK8编码（有三种类型：标准BASE64编码、URLBase64编码、MIMEBase64编码）
  		Encoder encoder = Base64.getEncoder();
  		byte[] bytes = "中文".getBytes("utf-8");
  		String encodeToString = encoder.encodeToString(bytes);//默认使用utr-8展示，将base64字节码转化为字符
  		System.out.println(encoder.encodeToString(bytes));
  //JDK8解码
  		Decoder decoder = Base64.getDecoder();
  		byte[] decode = decoder.decode(encodeToString);//默认使用utf-8进行Base64字符转化为字节码
  		System.out.println(new String(decode));
  
  //commons-codec解码（和JDK8没有什么区别）
  		Base64 base64 = new Base64();
  		byte[] encode = base64.encode("sdfa".getBytes());
  		String encodeToString = base64.encodeToString(encode);
  		System.out.println(encodeToString);
  ````

- Hex编码

   和Base64编码作用类似，将不读的二进制数据，转化为可显示的字符串数据；

   Hex编码原理：

   	将每个二进制字节一分为二，每四位二进制数转化位16进制（高四位全部补0）；然后将两个16进制数使用项目默认编码转化位英文字符

  ````java
   // 编码
   byte data[] = "A".getBytes("UTF-8");
   char[] encodeData = Hex.encodeHex(data);//获取Hex编码的字符数组
   String encodeStr = Hex.encodeHexString(data);//将Hex字符数组转化为字节数组，然后使用UTF-8编码，转化为字符
  
  // 解码
  byte[] decodeData = Hex.decodeHex(encodeData);//Hex编码的字符数组进行hex解码，获得字节数组
  System.out.println(new String(decodeData, "UTF-8"));
  ````

- MD5加密（Message-Digest Algorithm 5 信息摘要算法）

  MD5加密算法，特点：

  1、任意长度明文，加密后会得到固定长度（128位二进制数据，即16个字节）的MD5值

  2、不可逆加密算法，没有解密密钥，因此不会存在密钥丢失，而被破解；但是相应的加密速度变慢

  ````java
  byte[] md5 = DigestUtils.md5("1231213");//使用md5加密，将明文转化位16个字节的数组
  String md5Hex = DigestUtils.md5Hex("1231213");//使用md5加密，将明文转化位16个字节的数组；然后在使用MD5字符转化规则（将16个字节转化为32个字符）
  ````

  **MD5字符转化规则：**

  1、将字节转化为十进制，返回在-256~255

  2、然后分别取16的模、取16的商，对于匹配一个16个字符的数组 {"0","1","2","3","4","5","6","7","8","9","a","b","c","d","e","f"}

  3、最后将32个字符拼接为字符串

  **MD5加密算法用途：**

  1、作为**数字签名**，检验文件是否被篡改。由于MD5会通过数据计算出一个128位二进制，因此当文件传输时，加上一个MD5值，当接收方收到文件时，可以对文件进行MD5加密，比较得到两个MD5值，来判断文件是否被更改

  2、对数据库明文进行加密，由于不可以，因此无法通过数据库管理员身份之间获取加密信息；只能通过明文转化后和MD5对比，来确定明文是否匹配

- SHA加密（Secure HashAlgorithm 安全哈希算法）

和MD5非常相似，也是不可逆加密算法。但是加密后的数据长度为160位，因此更加安全；但是加密速度会慢一点

````java
DigestUtils.sha1("adff");
String sha1Hex = DigestUtils.sha1Hex("adff");
````

- DES加密（Data Encryption Standard 数据加密标准）

  是最常用的一种对称式加密算法，通过密钥进行加密，因此只要知道密钥，就可以将密文转化为原数据，并且加密速度很快

  由于只要直到密钥，就能够对密文进行破解，因此使用此算法时，需要保证密钥的安全性。一般情况下，对称式和非对称式加密一起使用：

  **数据加密使用对称式加密算法，保证加密速度，提高性能；然后在使用非对称式算法来加密管理密钥，防止密钥被破解**

- URL编码

  ​	在HttpUrl中，并不支持中文字符，并且对于有些字符：=、&等，都会影响url实际功能的使用，因此需要进行额外编码，然后进行传输；

  url默认支持字符：英文字符、数字、和一些保留字符;其他字符进行传输时都需要进行url编码 

  URL编码原理：

  ​	1、首先将字符转化为Unicode编码，然后在使用UTF-8转化为字节码；

  ​	2、根据utf-8字节码的个数（1-6个字节），把每个字节（8bit）二进制转化为2bit的16进制数

  ​	3、每个16进制数前面添加%，组合形成url编码

  ​		因此URL编码，理论上进行使用了UTF-8进行编码完成的；Tomcat在进行解析url时，会先将url编码转化为utf-8的二进制数据，此时就需要使用使用UTF-8进行解码；因此解析url中文乱码时，一定只能使用UTF-8进行中文解码

  浏览器或者jsAJAX请求url时，就会自动封装调用url编码，保证url的正确性

  ````java
  //JDK net包中，提高URLEncoder、URLDecoder来进行编码、解码（必须指定为utf-8）
  URLEncoder.encode(url,"utf-8");
  URLDecoder.decode(s, "utf-8");
  
  //commons-codec提高URLCodec来进行编码、解码（默认使用utf-8）
  URLCodec urlCodec = new URLCodec();
  urlCodec.encode(url);
  urlCodec.decode(url);
  ````


## 7、常用框架使用：

## 1、单元测试

​		**java单元测试是最小的功能单元测试代码，即针对单个java方法的测试（java的最小单位为方法）**

### main方法进行测试的缺点：

- 只能有一个main（）方法，不能把所有测试代码按一个个方法分开
- 无法打印出测试结果和预期结果

### 单元测试的优点：

- 可以打印测试报告
- 可以进行自动测试，进行执行结果的判断
- 可以在一个类中，书写多个测试方法，并且每个单元测试都可以独立执行

### JUnit4框架，java最常用的测试框架：

1、JUnit4框架有标准规定，测试类必须在src/test/java的目录中；maven项目就满足该目录结果

并且，**测试类不能使用Test命名，否则会出现导入的org.junit.Test类名冲突**

2、编写一个测试方法，只需要使用@Test注解，并且保证方法满足以下条件：

- 无返回值
- 无参数
- public

````java
@Test
public void Test(){
    
}
````

2、对单元测试进行断言（Assertion），即判定程序的执行结果是否符合预期，从而通过测试

使用assertEquals（obj，obj）、、、等方法来对结果进行断言

3、@Before、@After、@BeforeClass、@AfterClass使用，用于指定单元测试执行前后的代码（如资源初始化和释放）

**测试类执行单元测试的过程：**

- 初始化类的静态变量，执行一次@BeforeClass方法（该方法必须满足：无返回值、无参数、静态方法、public）
- 初始化类的成员变量，执行测试类的构造方法
- 执行@Before方法（和@Test方法一致）；然后执行@Test方法；最后执行@After方法
- 当执行测试类时，会自动从上到下执行所有@Test方法，因此相应也会多次执行初始化类的成员变量，执行测试类的构造方法、@Before和@After方法
- 最后在执行一次@AfterClass方法

**在执行测试类时，每一次执行@Test方法前，都是重新创建了一个测试类实例，因此测试类中的非静态成员变量，不能在每个@Test中共享，而是每次都为初始值**

**因此在多个@Test方法需要使用某个资源时，为了减少资源初始化和释放时间，一般将其初始化代码和释放代码放在@BeforeClass方法和@AfterClass中**

4、异常测试和超时测试；

@Test注解提供expected属性，用于指定异常类的Class对象，当未抛出该异常类型时，则测试失败；反之测试成功

@Test注解提供timeout属性，用于指定测试方法执行时间，单位为毫秒；当测试方法执行时间超出该值时，测试失败

5、忽略测试：

@Ignore，注解在已被@Test注解的方法上，用于在执行测试类时，忽略被注解的@test方法

6、使用不同运行器进行测试：

@RunWith（Calss），指定该测试类单元测试使用的运行器，如springboot测试类注解为@RunWith(SpringRunner.class)

不同的运行器，单元测试的规则和注解也有所不同，Junit默认运行器为 org.junit.runner.Runner

## 2、日志框架

推荐使用Slf4j+Logback,相对于Log4j有更多的优点、特性和性能

### **1、日志级别：**

| 日志级别 | 描述                                               |
| -------- | -------------------------------------------------- |
| OFF      | 关闭：最高级别，不输出日志。                       |
| FATAL    | 致命：输出非常严重的可能会导致应用程序终止的错误。 |
| ERROR    | 错误：输出错误，但应用还能继续运行。               |
| WARN     | 警告：输出可能潜在的危险状况。                     |
| INFO     | 信息：输出应用运行过程的详细信息。                 |
| DEBUG    | 调试：输出更细致的对调试应用有用的信息。           |
| TRACE    | 跟踪：输出更细致的程序运行轨迹。                   |
| ALL      | 所有：输出所有级别信息。                           |

一般情况下，使用 ERROR、WARN、INFO 、DEBUG四种级别

### 2、logback框架配置

1、java -Dlogback.configurationFile= /path/mylogback.xml  ;通过指定java执行命令，来加载自定义的logback配置文件

2、当没有指定logback.configurationFile参数时，默认在calsspath下按顺序查找：

- logback.groovy
- logback-test.xml
- logback.xml
- JDK6.0以上，自动查找com.qos.logback.classic.spi.**Configurator**接口的第一个实现类

 3、都没有找到时，使用**ch.qos.logback.classic.BasicConfigurator**类默认logback配置，将日志在控制台输出

logback.xml常用配置

````xml
<?xml version="1.0" encoding="UTF-8"?>
 <!-- scan 配置文件改变后，是否重新加载
      scanPeriod 检测配置文件是否发送改变的时间周期，默认为毫秒，可以自定义单位
      debug 是否打印logback内部日志（一般不需要）
  -->
<configuration scan="ture" scanPeriod="60 seconds" debug="false">
    <!-- 定义参数常量，用于在日志组件中使用，达到全局变量的作用
         log.level  日志打印最低级别
         log.maxHistory 日志文件保存时间
         log.filePath 日志文件保存根目录
         log.pattern 日志打印格式 :%d时间、{yyyy-MM-dd HH:mm:ss:SSS}时间格式、%thrad线程名、
         %-5level 5个字符宽度展示日志等级、%logger 打印日志的类限定名、&%msg日志信息 、%n换行符
          -->
    <property name="log.level" value="debug"/>
    <property name="log.maxHistory" value="30"/>
    <property name="log.filePath" value="${catalina.base}"/> <!-- ${catalina.base}为Tomcat根目录 -->
    <property name="log.pattern" value="%d{yyyy-MM-dd HH:mm:ss:SSS} [%thead] %-5level %logger{50} - %msg%n"/>
    
    <!-- 定义日志在控制台输出的组件 -->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
    <!-- 日志输出格式 -->
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
    </appender>

   <!-- 定义日志在文件中输出的组件
     使用多个组件来定义不同等级的日志文件输出，一般使用三个等级：DUBUG、INFO、ERROR
     并且都使用RollingFileAppender作为组件（文件输出日志内容，并可以设置滚动策略，即达到一定条件后，生成新日志文件 
   FileAppender 一般不使用，会将所有日志输出在一个日志文件中-->
    <appender name="debugAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 文件路径 -->
        <file>${log.filePath}/debug.log</file>
        <!-- 滚动策略，TimeBasedRollingPolicy,每天生成一个文件，指定文件名、文件保存时间 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.filePath}/dubug/dubug.%d{yyyy-MM-dd}.log.gz}</fileNamePattern>
            <MaxHitory>${log.maxHistory}</MaxHitory>
        </rollingPolicy>
        <!-- 日志输出格式 -->
        <encoder>
            <pattern>${log.pattern}</pattern>
        </encoder>
        <!-- 使用组件日志等级过滤器，该组件只作为DUBUG等级日志的输出源 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DUBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
     
    <!-- 定义全局logger、所有类的日志都使用下面的appender进行输出
                  当LoggerFactory.getLogger(name/class)无法找到对于loggin名时，使用root全局设置输出 -->
    <root level="info">
        <appender-ref ref="consoleAppender"/>
    </root>
    
    
    <!-- 定义logger，指定logger名，用于匹配LoggerFactory.getLogger(name/class)；
    class则匹配其类对应的包名.
    logger默认是继承使用root中的Appender组件，因此当root指定控制台输出后，logger中不需要指定;
        当然也可以通过additivity="false",不继承root中的Appender组件-->
    <logger name="testLogger" level="${log.level}">
        <appender-ref ref="debugAppender"/>
    </logger>
   
</configuration>
````

### 3、logback启动原理

- 使用slf4j接口的好处：基本所有日志框架，都可以作为其实现层，因此我们可以面向该接口进行编程，通过切换slf4j的实现，来简单完整实现底层日志框架的切换

- logback启动过程：

  调用slf4接口的方法：LoggerFactory.getLogger(Test.class)；此时slf4j调用org.slf4j.StaticLoggerBinder进行初始化，就会扫描项目中所有`org.slf4j.StaticLoggerBinder`的具体实现；

  而logback-classic中就定义了StaticLoggerBinder的实现方法，此时就会加载logback.xml

### 4、logback日志使用要点

- 定义logger变量

````java
private static final Logger logger = LoggerFactory.getLogger(XX.class);
要确保一个类中，只有一个logger对象（静态成员常量），从而减少对象的内存占用
````

- 在生产环境中，应该禁止debug级别日志的输出

  由于logger会继承root的输出源，因此一般需要添加additivity="true"，放在日志重复输出

- error日志信息输出时，应该携带异常信息，帮助分析异常原因

````java
		try {
			int a= 1/0;
		} catch (Exception e) {
			log.error("test发生异常，异常信息："+e.getMessage(),e);
		}
````

- 减少日志打印对应用程序性能的损耗

  虽然我们在生产环境下，会提高日志级别的打印，减少不必要的日志打印；但是日志中的字符串拼接还是会执行，这样会白白浪费性能，因此需要以下手段来进行这样情况的发生：

  1、使用条件判断，判断当前logger对象的需要输出的日志级别，再对日志进行打印后者忽略

  ````java
  	if (log.isErrorEnabled()) {
  		log.error("test发生异常，异常信息："+e.getMessage(),e);
  	}
  ````

  2、使用占位符，来减少字符串拼接时，String对象的创建；使用占位符时，只有需要打印该字符串时，才会进拼接（有些日志框架不支持占位符,logback支持）

  ````java
  	int id=100;
  	String user="yh"; 
  	log.debug("错误，id：{},user:{}",id, user);
  ````

- 日志文件命名：推荐使用projectName_logName_logType.log

## 3、lombok

lombok是简化java开发代码编写的一个java库。通过添加注解的方式，来简化java代码，提高开发人员的开发效率

### 常用注解：

@Data：自动为该类所有非静态属性添加setter/getter、equals、equal、hashCode、toString方法；对于final属性，则不会生产setter方法；

@Slf4j：自动创建该类的一个静态常量日志对象：log

@NonNull 注解在字段或者构造器的方法形参中；注解在字段上时，需要配合@Data使用，会自动生成带有该字段参数的构造函数，并且参数为null时，会抛出NPE异常；当注解在方法上时，会生成对参数进行非null的检测，若为null，则抛出NPE异常

@Cleanup 注解在Java资源对象（io流、数据库操作流。。）上，自动添加try—finally{}，在其中关闭资源

@Getter/Setter 注解在类或者字段上，注解在类则为全部字段；注解在字段则为单个字段

@ToString 注解在类上，自动生成toString方法

@EqualsAndHashCode 注解在类上，自动生成hashCode和equal方法

@NoAragsConstructor 注解在类上，生成无参构造方法

### lombok实现原理：

实际上是在javac执行之前，添加对应Lombok的语法树，并进行修改；之后添加到class文件的相应节点（代码块）

## 4、cors解决跨域

````java
@WebFilter("/*")
public class CorsFilter implements Filter {

	@Override
	public void destroy() {
	}

	@Override
	public void init(FilterConfig arg0) throws ServletException {
	}

	@Override
	public void doFilter(ServletRequest arg0, ServletResponse arg1, FilterChain arg2)
			throws IOException, ServletException {

		HttpServletResponse response = (HttpServletResponse) arg1;
		HttpServletRequest request = (HttpServletRequest) arg0;

		// 当浏览器发出跨域的请求时，会自动添加Origin的请求头，vluae为请求所在项目的地址（ip+端口）
		// 此时服务器端就需要判断该地址是否允许进行访问，允许就将该地址放入响应中，让浏览器判断是否允许跨域
		// response.setHeader("Access-Control-Allow-Origin", "*");也可以直接设置为*，表示允许所有跨域请求
		response.setHeader("Access-Control-Allow-Origin", request.getHeader("Origin"));

		// 对于简单的跨域请求（请求方式为：get、head、post；
		// content-Type为：text/plain、multipart/form-data、appliaction/x-www-form-urlencoded三者之一等条件的请求）
		// 只需要指定响应头Access-Control-Allow-Origin为源地址或者*，让浏览器不拦截跨域请求结果

		// 对于复杂请求，需要指定以下header，来限制允许跨域请求的条件，让浏览器选择性拦截不合理的跨越请求；
		// 浏览器会先发出一个预检请求（OPTIONS），避免跨域请求对服务器的用户数据产生未预期的影响。
		// 并告知实际请求需要使用的方法、请求头，然后根据服务器允许的请求条件做匹配，可以则发出实际请求

		// 因此跨域限制的判断，全是通过浏览器来完成，浏览器会根据请求是否跨域，来使用不同的请求方式（简单请求、复杂请求）和添加必须的请求头
		// 服务器只是通过设置Access-Control-Allow-Origin等字段，来告诉浏览器，是否拦截该跨越请求的结果

		// 允许请求发送cookie，但此时Access-Control-Allow-Origin就不能设置为*
		response.setHeader("Access-Control-Allow-Credentials", "true");
		// 允许请求方式、请求头字段
		response.setHeader("Access-Control-Allow-Methods", "POST, PUT, GET, OPTIONS, DELETE");
		response.setHeader("Access-Control-Allow-Headers",
				"Origin, No-Cache, X-Requested-With, If-Modified-Since, Pragma, Last-Modified, Cache-Control, Expires, Content-Type, X-E4M-With,Authorization,Token");
		response.setHeader("Access-Control-Max-Age", "5000");// 预检请求有效期
		arg2.doFilter(arg0, arg1);

	}

}
````

## 5、jackson

### jackSon有三种处理JSON的方式：

1、通过数据绑定，来完成json与java对象的转化

```java
		//Jackson主要类,用于实现java对象和json字符串的转化；
		//内部使用JsonParser和JsonGenerator的实例实现JSON实际的读/写（一般重复使用）
		ObjectMapper objectMapper = new ObjectMapper();
		
		//json字符串反序列化为指定对象
		String json ="{\"name\":\"YH\",\"id\":\"54321\"}";
		HashMap<String, String> hashMap = new HashMap<String, String>();
		try {
			hashMap = objectMapper.readValue(json, HashMap.class);
			System.out.println(hashMap);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
		
		//hashMap序列化为json字符串
		try {
			String writeValueAsString = objectMapper.writeValueAsString(hashMap);
			System.out.println(writeValueAsString);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
```

objectMapper还提供多个重载的writeValue（）方法；参数为文件/输出流/jcaksonGenerator（json生成器）和需要转化为json的值；这样就可以实现将json字符串写入到文件、输出流、和json生成器中

2、使用Jackson的树模型，用于获取较大json中的某个字段，不需要解析整个json，因此减少不必要的性能损失

````java
		// json字符串转化为jsonNode（json节点，每个节点都有自己对应的值）
		String json = "{\"name\":\"YH\",\"id\":\"54321\"}";
		try {
			JsonNode readTree = objectMapper.readTree(json);
			JsonNode jsonNode = readTree.get("name");
			String asText = jsonNode.asText();
			System.out.println(asText);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
````

当然也可以使用树模型来创建json字符串，但是操作繁琐

````java
		ObjectNode rootNode = objectMapper.createObjectNode();
		rootNode.put("name","yh");//json节点插入字段
		rootNode.put("id","123");
		ObjectNode object = objectMapper.createObjectNode();
		rootNode.set("objcet",object);//json节点插入对象
		object.put("hh", "1332");//json子节点插入字段
		//将json整个树转化为json字符串
		try {
			String writeValueAsString = objectMapper.writeValueAsString(rootNode);
			System.out.println(writeValueAsString);
		} catch (JsonProcessingException e) {
			e.printStackTrace();
		}
````





3、使用Jackson流式API，使用JsonParser、JsonGenerator进行读写

JacksonAPI是一套比较底层的API，因此速度最快，但使用起来也麻烦

````java
//创建json字符串
		JsonFactory jsonFactory = new JsonFactory();
		StringWriter stringWriter = new StringWriter();
		JsonGenerator generator = jsonFactory.createGenerator(stringWriter);
		generator.writeStartObject();//开始生成json对象
//		generator.writeStartArray();//开始生成json数组
//		json键值对
		generator.writeNumberField("id", 123);
		generator.writeStringField("name", "dd");
		
//		json数组
		generator.writeFieldName("test");
		generator.writeStartArray();
		generator.writeString("msg1");
		generator.writeString("msg2");
		generator.writeEndArray();
		
		//json对象
		generator.writeFieldName("object");
		generator.writeStartObject();
		generator.writeNumberField("id", 1234);
		generator.writeStringField("name", "ddd");
		generator.writeEndObject();
		
		generator.writeEndObject();
		generator.close();
		System.out.println(stringWriter.toString());

//解析json字符串
		JsonFactory jsonFactory = new JsonFactory();
		String json1 = "{\"name\":\"YH\",\"id\":\"54321\"}";
		JsonParser parser = jsonFactory.createParser(json1);
		while (parser.nextToken() != JsonToken.END_OBJECT) {
			String FieldName = parser.getCurrentName();
			if ("name".equals(FieldName)) {
				parser.nextToken();
				System.out.println(parser.getText());
			}
			if ("id".equals(FieldName)) {
				parser.nextToken();
				System.out.println(parser.getText());
			}
		}
````





### jackson常用注解，以及使用场景

1、@JsonInclude，注解在类或属性上；常用value值有：JsonInclude.Include.NON_EMPTY（属性为空、null时，不参与序列化）、JsonInclude.include.NON_NULL(属性为null时，不参与序列化)

使用场景：当该属性值为null时，不需要传递给前端

2、@JsonIinore，注解在属性上，告诉jackson在进行json序列化或反序列化时，忽略该属性（转化为java对象时，该属性为null；转化为json字符串时，没有该字段）

@JsonIgnoreProperties，注解在类上，和@JsonIinore一样，但是可以指定多个属性value={“id“，”name“}

使用场景：对于某些实体类外键属性（用于两个表数据关联的字段），不需要传递给前端

### Jackson关键类ObjectMapper属性设置

````java
ObjectMapper objectMapper = new ObjectMapper();
//对objectMapper序列化的json字符串进行格式化处理，展示更加清晰（但线上不推荐使用，会增加json字符串的大小）
objectMapper.configure(SerializationFeature.INDENT_OUTPUT, true);

//Map序列化生成的json字符串时，按key值进行自然排序
objectMapper.configure(SerializationFeature.ORDER_MAP_ENTRIES_BY_KEYS, true);

//所有java对象序列化生成json字符串时，按key值进行自然排序
objectMapper.configure(MapperFeature.SORT_PROPERTIES_ALPHABETICALLY,true);

//生成json字符串时（序列化）忽略为null、空的字段
objectMapper.setSerializationInclusion(Include.NON_EMPTY);

//生成json字符串时（序列化），修改对应属性映射的字段名（Test中为name，json中为Test_name）
		objectMapper.setPropertyNamingStrategy(new PropertyNamingStrategy() {
            //对于共有成员变量（直接可以通过对象调用）
//			@Override
//			public String nameForField(MapperConfig<?> config, AnnotatedField field, String defaultName) {
//				if (field.getFullName().equals("com.yh.Test#name")) {
//					return "Test_name";
//				}
//				return super.nameForField(config, field, defaultName);
//			}
            
            //对于私有成员变量（需要通过get（）方法获取）
			@Override
			public String nameForGetterMethod(MapperConfig<?> config, AnnotatedMethod method, String defaultName) {
				if (method.getAnnotated().getDeclaringClass().equals(Test.class)&&defaultName.equals("name")) {
					return "Test_name";
				}
				return super.nameForGetterMethod(config, method, defaultName);
			}
		});
````