### Bean生命周期

Bean生命周期描述的是Spring中一个Bean创建和销毁过程中所经历的步骤，其中Bean创建过程是重点，程序员可以利用Bean生命周期机制对Bean进行自定义加工。

BeanDefinition Bean定义 ->构造方法推断选出一个构造方法 -> 实例化 构造方法反射得到对象 -> 属性填充 ->初始化 ->初始化后（AOP、生成代理对象）

#### BeanDefinition

表示Bean定义，它定义了某个bean的类型，Spring就是利用BeanDefinition来创建Bean的，比如需要利用BeanDefinition中beanClass属性确定Bean类型，从而实例化出来对象

#### 构造方法推断

一个bean中可以有多个构造方法，此时需要Spring来判断到底使用哪个构造方法，通过构造方法推断之后确定一个构造方法后，就可以利用构造方法实例化得到一个对象。

#### 实例化

通过构造方法反射得到一个实例化对象，在Spring中，可以通过BeanPostProcessor机制对实例化对象进行干预

### @Autowired是什么

@Autowired表示某个属性是否需要进行依赖注入，可以写在属性和方法上。注解中的required属性默认为true，表示如果没有对象可以注入抛出异常。

@Autowired加在某个属性上，Spring在进行Bean的生命周期过程中，在属性填充这一步会基于实例化出来的对象，对该对象中加了@Autowired的属性自动给属性赋值。

Spring会先根据属性的类型去Spring容器中找出该类型所有的Bean对象，如果出来多个，则再根据属性名字从多个中再确定一个。如果require属性true。并且根据属性信息找不到对象直接抛出异常。

@Autowired表示一个属性是否需要进行依赖注入，可以使用在属性、普通方法上、构造方法上。注解中的required属性默认是true，如果没有对象可以注入到属性，则会报出异常；

@Autowired加在某个属性上，spring会从ioc容器中找到bean对象注入到属性上，如果找到多个该类型的Bean对象，则再根据属性的名字从多个Bean对象中确认一个；

@Autowired写在set()方法上，在spring会根据方法的参数类型从ioc容器中找到该类型的Bean对象注入到方法的行参中，并且自动反射调用该方法(被@Autowired修饰的方法一定会执行)，所以一般使用在set方法中、普通方法不用；

@Autowired使用在构造方法中：根据构造方法的形参、形参名，从ioc容器中找到该类型的Bean对象，注入到构造方法的形参中，并且执行该方法；

@Autowired注解在进行依赖注入的时候需要指定bean的时候，和@Qualifier注解一起使用使用@qualifier注解指定名称

### @Resource

@Resource注解与@Autowired注解类似，也是用来进行依赖注入，@Resource是Java层面提供注解。

`@Resource`属于 JDK 提供的注解，默认注入方式为 `byName`。如果无法通过名称匹配到对应的 Bean 的话，注入方式会变为`byType`。

`@Resource` 有两个比较重要且日常开发常用的属性：`name`（名称）、`type`（类型）。

### @Value

@Value("Tony")

直接将字符串Tony赋值给属性，如果属性类型不是String，无法进行类型转换

@Value("${Tony}")

将会把${}中的字符串当作key，从properties文件中找出对应value赋值给属性，如果没有找到则会把${Tony}当作普通字符串注入给属性

@Value("#{Tony}")

会将#{}中的字符串当作Spring表达式进行解析，Spring会把"Tony"当作beanName，并从Spring容器中找对应bean，如果找到则进行属性注入，没找到则报错

### FactoryBean

FactoryBean是Spring所提供的一种较灵活的创建Bean的方式，可以通过实现FactoyBean接口中的getObject()方法来返回一个对象，这个对象就是最终Bean对象

FactoryBean接口中的方法

Object getObject() 返回的是Bean对象

boolean isSingleton() 返回的是否为单例Bean对象

Class getObjectType() 返回的是Bean对象的类型

### ApplicationContext

ApplicationContext是比BeanFactory更加强大的Spring容器，它既可以创建bean，获取bean，还可以国际化、事件广播、获取资源等BeanFactory不具备的功能

### BeanPostProcessor

BeanPostProcessor是Spring所提供的一种扩展机制，可以利用该机制对Bean进行定制化加工。在Spring底层源码实现中，也广泛的用到该机制，BeanPostProcessor通常也叫做Bean后置处理器。

BeanPostProcessor在Spring中是一个接口，我们定义一个后置处理器，就是提供一个类实现该接口，在Spring中还存在一些接口继承了BeanPostProcessor，这些子接口是在BeanPostProcessor的基础上增加了一些其他的功能。

相关方法

postProcessBeforeInitialization()：初始化前方法，表示可以利用这个方法来对Bean在初始化前进行自定义加工。

postProcessAfterInitialization(): 初始化后方法，表示可以利用这个方法对Bean在初始化后进行自定义加工

InstantiationAwareBeanPostProcessor

BeanPostProcessor的一个子接口

postProcessBeforeInstantiation() :实例化前

postProcessAfterInstantiation(): 实例化后

postProcessProperties() 属性注入后

### AOP工作原理

- Spring生成bean对象时，先实例化出来一个对象，也就是target对象

- 对target对象进行属性填充

- 在初始化后步骤中，会判断target对象有没有对应切面

- 如果有切面就表示当前target对象需要进行AOP

- 通过Cglib或JDK动态代理机制生成一个代理对象，作为最终bean对象

  