### lambda表达式

一种新的语法，允许通过表达式来代替功能接口

基本语法：

(parameters)  -> expression

(parameters)  -> {statements;}

- 可以用来简化匿名内部类，比如常见的new Runnable{  run(){…}   }，就可以直接缩写成 ()  -> … ，在编写函数式接口时非常方便（什么叫函数式接口？你看过了）

- 与Streams结合：Streams是对集合的包装，可以结合lambda表达式使用，看下面的例子

  ```java
  System.out.println("下面是月薪超过 $1,400 的PHP程序员:")  
  //filter函数接收一个过滤器去对stream中的内容进行过滤，然后又返回一个stream对象，这个过滤器（过滤方法）接收一个对象，然后需要返回一个布尔表达式，这就很适合交给lambda表达式完成，（实际上是实现了Predicate接口的evaluate方法）
  phpProgrammers.stream()  
            .filter((p) -> (p.getSalary() > 1400))  
            .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
  ```

  也可以写好一些过滤器以复用

  ```java
  // 定义 filters  
  Predicate<Person> ageFilter = (p) -> (p.getAge() > 25);  
  Predicate<Person> salaryFilter = (p) -> (p.getSalary() > 1400);  
  Predicate<Person> genderFilter = (p) -> ("female".equals(p.getGender()));  
    
  System.out.println("下面是年龄大于 24岁且月薪在$1,400以上的女PHP程序员:");  
  phpProgrammers.stream()  
            .filter(ageFilter)  
            .filter(salaryFilter)  
            .filter(genderFilter)  
            .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));  
    
  // 重用filters  
  System.out.println("年龄大于 24岁的女性 Java programmers:");  
  javaProgrammers.stream()  
            .filter(ageFilter)  
            .filter(genderFilter)  
            .forEach((p) -> System.out.printf("%s %s; ", p.getFirstName(), p.getLastName()));
  ```

- Stream中还有一些比较重要的编程概念

  - map:对每个元素进行操作
  - reduce：将所有元素的值结合起来



#### 几个需要注意的函数式接口（他们都跟stream有关）

Consumer（消费型）：用于输出信息`public void accept( T t)`

Supplier（供给型）：一个容器，用于提供T类型的数据 `public T get()`

Predicate（谓词型）：就是用来做判断的，“是”或者“不是”，`public boolean test( T t)`

Function（功能型）：转换作用，将输入数据转换成另一种形式的输出  `public T apply (S s)`，比如传入字符串传出字符串长度



### 接口的默认方法和静态方法

默认方法会被重写

静态方法不会被继承，只能通过接口名调用



### 方法引用

用来直接访问类或者实例已经存在的方法或构造方法

在Lambda表达式中只是执行一个方法调用时

```java
public void introduce() {
    String[] strArray ={"a", "c", "b"};
    //在学完lambda表达式后，如果你想创建一个Comparator实例，你可能会用以下方法：
    Arrays.sort(strArray, (s1,s2)->s1.compareTo(s2));
    //你仔细看就会发现整个lambda体其实就只是在调用String类的compareTo()方法而已
    //因此我们可以使用方法引用的方式来进行简写
    Arrays.sort(strArray,String::compareTo);
}
```

- 方法引用 ，形式有三种： 1、实例::实例方法名   2、 类名::静态方法名   3、类名::实例方法名
- 构造器引用 ，形式有一种： 类名::new
- 数组引用  ，形式有一种：类型[]::new



#### 方法的参数名可以获取到啦！

```java
public class ParameterNames {
    public static void main(String[] args) throws Exception {
        //利用反射特性
        Method method = ParameterNames.class.getMethod( "main", String[].class );
        for( final Parameter parameter: method.getParameters() ) {
            System.out.println( "Parameter: " + parameter.getName() );
        }
    }
}
```

在Java 8中这个特性是默认关闭的，因此如果不带**-parameters**参数编译上述代码并运行，则会输出如下结果：

```
Parameter: arg0
```

如果带**-parameters**参数，则会输出如下结果（正确的结果）：

```
Parameter: args
```



### 更好用的时间API

