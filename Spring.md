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