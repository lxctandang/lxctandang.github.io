# Lombok的使用学习

​		lombok是一个简单的java库，该工具可以通过注解的方式可以给你的类自动的添加一些方法，简化开发。比如编写POJO时需要为每个属性提供getter()和setter()方法，这些都可以使用Lombok注解来实现。**随便提一下就是，如果想要使用Lombok，在IDEA需要安装Lombok插件，同时IDEA设置开启支持注解的编译过程，然后在你的项目中导入依赖就可以用了。**

```
// 引入Lombok依赖。
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.10</version>
</dependency>
```

## Lombok的常用注解

@Setter、@Getter、@Data、@NoArgsConstructor、@AllArgsConstructor、@NonNull、@RequiredArgsConstructor、@ToString、@EqualsAndHashCode、@Builder

@Log      日志注解，注解在类上，可以使用日志注释对任何类进行注释，以是Lombok生成记录器字段，而且@Log有许多的变体。@CommonsLog 、@Flogger、@JBossLog、@Log、@Log4j、@Log4j2、@Slf4j、@XSlfj

@Value、@CleanUp

### 1.@Setter

该注解可以注释类，也可以注解属性，但是情况是有所不同的。

​	1.注解类时，可以为类中的每个（非final和非static）属性默认生成setXxx(Xxx xxx)方法；

​    2.注解属性时，可以为这个（非final）属性默认生成setXxx(Xxx xxx)方法；

注意的是：这个属性被关键字final和static修饰，可能不会自动生成Setter方法。

### 2.@Getter

该注解和@Setter一样，可以注解类，也可以注解属性，仍然情况有所不同。

​	1.注解类时，可以为类中的每个（非final和非static）属性默认生成getXxx(Xxx xxx)方法；

​	2.注解属性时，可以为这个（包括final和static修饰的）属性默认生成getXxx(Xxx xxx)方法；

### 3.@Log、@Slf4j

日志注解，注解在类上，日志可以分为多个级别 debug、info、warn、errorspringboot中默认级别为info级别。使用@Slf4j注解之后。就可以在程序的运行时，查看程序运行的状态。默认名字为log，如果想改可以例如: @Slf4j(topic="hello")。 

```
@RestController
@Slf4j
public class ScoreController {
    @RequestMapping("/log")
    public String printlnLog() {
        System.out.println("李小池的测试程序");
        log.debug("debug测试");//默认日志级别为info  
        log.info("info测试");
        log.warn("warn测试");
        log.error("error测试");
        return "启动成功";
    }
}
```

换句话说就是，以前我们想要查看程序的运行状态，需要在程序的多个地方插入System.out.println(“信息")输出到控制台的语句，但是这个方式，会和正常的数据杂糅在一起，不利于区分。现在可以使用日志，不仅设置了输出级别，对不同的级别的信息进行分类显示，而且可以和正常输出的数据更好的区分。

### 4.@AllArgsConstructor

该注解作用在类上，自动为该类提供一个全参数（不包括final和static修饰的属性）的构造方法，注意：默认不提供无参数构造方法。

### 5.@NoArgsConstructor

该注解作用在类上，自动为该类添加一个无参数的构造方法。

### 6.@EqualsAndHashCode

该注解用在类上，自动为该类提供equals（）和hashCode()。如果对象需要放进HashMap对象中，我们可以简单的为它的类添加这个注解。

### 7.@NonNull

注解在属性上或者是方法的入参上，还可以在方法的返回值上，用于非空检查。

注解作用在类的成员变量，可以和@RequiredArgsConstructor搭配使用。

如果注解在方法的入参上，会在方法体中最开始生成一个null检查（if语句），如果为空，抛出NullPointerException。

### 8.@RequiredArgsConstructor

该注解使用在类上，为该类生成一个构造方法--参数是该类中所有被@NonNull注解的变量（包括被fianl和修饰的变量）。

### 9.@ToString

注解在类上，为该类自动生成toString()方法。默认将所有非静态变量以key-value形式输出。但该注解提供三个属性可供配置：

- includeFieldNames：是否包含属性名称。默认为true，如果设置为false则只是将属性的值以Set的形式输出。
- exclude：排除指定字段
- callSuper：输出父类属性。注意：父类也要使用该注解或者提供有toString()方法，否则输出的是父类的内存地址。

### 10.@Data

注解在类上，该注解是最常用的注解，它结合了@ToString，@EqualsAndHashCode， @Getter和@Setter。本质上使用@Data注解，类默认有@ToString和@EqualsAndHashCode以及每个非final和static字段都有@Setter和@Getter。虽然@Data注解会生成构造方法，但是有问题，建议使用构造方法的注解。

### 11.@Builder

使用构造者模式，在类的内部定义一个静态的内部类，用于创建当前对象。