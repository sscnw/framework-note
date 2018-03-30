#Spring中Bean的配置
##IOC&DI概述
- IOC(Inversion of Control)：其思想是反转资源获取的方向. 传统的资源查找方式要求组件向容器发起请求查找资源. 作为回应, 容器适时的返回资源. 而应用了 IOC 之后, 则是容器主动地将资源推送给它所管理的组件, 组件所要做的仅是选择一种合适的方式来接受资源. 这种行为也被称为查找的被动形式
- DI(Dependency Injection) — IOC 的另一种表述方式：即组件以一些预先定义好的方式(例如: setter 方法)接受来自如容器的资源注入. 相对于 IOC 而言，这种表述更直接
##配置 bean
- 配置形式：基于 XML 文件的方式  ；基于注解的方式
- Bean 的配置方式：通过全类名（反射）、通过工厂方法（静态工厂方法 & 实例工厂方法）、FactoryBean
- IOC 容器 BeanFactory & ApplicationContext 概述
- 依赖注入的方式：属性注入；构造器注入
- 注入属性值细节
- 自动转配
- bean 之间的关系：继承；依赖
- bean 的作用域：singleton；prototype；WE  B 环境作用域
- 使用外部属性文件
- spEL 
- IOC 容器中 Bean 的生命周期
- Spring 4.x 新特性：泛型依赖注入
##IOC前生
- 需求: 生成 HTML 或 PDF 格式的不同类型的报表.
	- 分离接口与实现
	![](imgs/20180330-183740.png)
	- 工厂设计模式
![](imgs/20180330-183805.png)
	- 采用反转控制
![](imgs/20180330-183815.png)
##在 Spring 的 IOC 容器里配置 Bean
- 在 xml 文件中通过 bean 节点来配置 bean
![](imgs/20180330-183923.png)

	- id：Bean 的名称。他哦你通过id来获取这个Bean
在 IOC 容器中必须是唯一的
若 id 没有指定，Spring 自动将权限定性类名作为 Bean 的名字
id 可以指定多个名字，名字之间可用逗号、分号、或空格分隔
	- class ：Bean的全类名。通过反射的方式在IOC容器中创建Bean的实例，所以在Bean中必须要带有一个无参的构造器
	
- Spring 容器
	- 在 Spring IOC 容器读取 Bean 配置创建 Bean 实例之前, 必须对它进行实例化. 只有在容器实例化后, 才可以从 IOC 容器里获取 Bean 实例并使用.
	- Spring 提供了两种类型的 IOC 容器实现. 
 	- BeanFactory: IOC 容器的基本实现.
	- ApplicationContext: 提供了更多的高级特性. 是 BeanFactory 的子接口.
	- BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身；ApplicationContext 面向使用 Spring 框架的开发者，几乎所有的应用场合都直接使用 ApplicationContext 而非底层的 BeanFactory
	- 无论使用何种方式, 配置文件时相同的.
- ApplicationContext
	- ApplicationContext 的主要实现类：
ClassPathXmlApplicationContext：从 类路径下加载配置文件
FileSystemXmlApplicationContext: 从文件系统中加载配置文件
	- ConfigurableApplicationContext 扩展于 ApplicationContext，新增加两个主要方法：refresh() 和 close()， 让 ApplicationContext 具有启动、刷新和关闭上下文的能力
	- ApplicationContext 在初始化上下文时就实例化所有单例的 Bean。
	- WebApplicationContext 是专门为 WEB 应用而准备的，它允许从相对于 WEB 根目录的路径中完成初始化工作
	![](imgs/20180330-184230.png)
	- 从 IOC 容器中获取 Bean
	调用 ApplicationContext 的 getBean() 方法
	![](imgs/20180330-111057.png)
	利用id定位到IOC中的bean
	```HelloWorld  helloWorld=（HelloWorld）ctx.getBean("id");```
	利用类型定位到IOC中的Bean,要求IOC中必须只有一个该类型的Bean。
	因为当配置文件中一个ioc配置了都为helloworld这个类型但是不同的两个Bean，编译就会出错
	```HelloWorld  helloWorld=ctx.getBean(HelloWorld.class);```
- 依赖注入的方式
	- 属性注入
		- 属性注入即通过 setter 方法注入Bean 的属性值或依赖的对象
		- 属性注入使用 <property> 元素, 使用 name 属性指定 Bean 的属性名称，value 属性或 <value> 子节点指定属性值 
		- 属性注入是实际应用中最常用的注入方式
		![](imgs/20180330-111255.png)
  	- 构造器注入
  		- 通过构造方法注入Bean 的属性值或依赖的对象，它保证了 Bean 实例在实例化后就可以使用
  		- 构造器注入在 <constructor-arg> 元素里声明属性, <constructor-arg> 中没有 name 属性
  		- 区分重载的构造器
  			- 按索引匹配入参：
  			![](imgs/20180330-111344.png)
  			- 按类型匹配入参：
  			![](imgs/20180330-111358.png)
	- 工厂方法注入（很少使用，不推荐）
- 注入属性值细节
	- 字面值
		- 字面值：可用字符串表示的值，可以通过 `<value>` 子节点（标签）或 value 属性进行注入。
		- 基本数据类型及其封装类、String 等类型都可以采取字面值注入的方式
		- 若字面值中包含特殊字符，如<>,可以使用 `<![CDATA[]]>` 把字面值包裹起来。
	- 引用其它 Bean
		- 组成应用程序的 Bean 经常需要相互协作以完成应用程序的功能. 要使 Bean 能够相互访问, 就必须在 Bean 配置文件中指定对 Bean 的引用
		- 在 Bean 的配置文件中, 可以通过` <ref>` 元素或 ref  属性为 Bean 的属性或构造器参数指定对 Bean 的引用. 
		- 也可以在属性或构造器里包含 Bean 的声明, 这样的 Bean 称为内部 Bean。不能被外部引用，只能在内部使用。
		![](imgs/20180330-112912.png)
	- 内部 Bean
		- 当 Bean 实例仅仅给一个特定的属性使用时, 可以将其声明为内部 Bean. 内部 Bean 声明直接包含在 `<property>` 或` <constructor-arg>` 元素里, 不需要设置任何 id 或 name 属性
		- 内部 Bean 不能使用在任何其他地方
	- 注入参数详解：null 值和级联属性
		- 可以使用专用的 `<null/> `元素标签为 Bean 的字符串或其它对象类型的属性注入 null 值，但是这个意义不大，因为可以不用赋值，就是null
		- 和 Struts、Hiberante 等框架一样，Spring 支持级联属性的配置。
		- 级联配置时，属性需要先初始化后才可以级联属性赋值，和Strut2不同。
		- 集合属性
			- 在 Spring中可以通过一组内置的 xml 标签(例如: `<list>`, `<set>` 或` <map>`) 来配置集合属性.
			- 配置 java.util.List 类型的属性, 需要指定` <list>`  标签, 在标签里包含一些元素. 这些标签可以通过 `<value>` 指定简单的常量值, 通过 `<ref>` 指定对其他 Bean 的引用. 通过`<bean>` 指定内置 Bean 定义. 通过 `<null/>` 指定空元素. 甚至可以内嵌其他集合.
			- 数组的定义和 List 一样, 都使用 `<list>`
			- 配置 java.util.Set 需要使用 `<set> `标签, 定义元素的方法与 List 一样.
			- Java.util.Map 通过` <map> `标签定义, `<map>` 标签里可以使用多个 `<entry>` 作为子标签. 每个条目包含一个键和一个值. 
				- 必须在` <key>` 标签里定义键
				因为键和值的类型没有限制, 所以可以自由地为它们指定` <value>`, `<ref>`, `<bean> `或` <null>` 元素. 
				- 可以将 Map 的键和值作为` <entry>` 的属性定义: 简单常量使用 key 和 value 来定义; Bean 引用通过 key-ref 和 value-ref 属性定义
			- 使用 `<props>` 定义 java.util.Properties, 该标签使用多个 `<prop>` 作为子标签. 每个 `<prop>` 标签必须定义 key 属性.
			-  使用 utility scheme 定义集合
				- 使用基本的集合标签定义集合时, 不能将集合作为独立的 Bean 定义, 导致其他 Bean 无法引用该集合, 所以无法在不同 Bean 之间共享集合.
				- 可以使用 util schema 里的集合标签定义独立的集合 Bean. 需要注意的是, 必须在 `<beans> `根元素里添加 util schema 定义
			- 使用 p 命名空间为bean的属性赋值
				- 为了简化 XML 文件的配置，越来越多的 XML 文件采用属性而非子元素配置信息。
				- Spring 从 2.5 版本开始引入了一个新的 p 命名空间，可以通过` <bean>` 元素属性的方式配置 Bean 的属性。			
				- 使用 p 命名空间后，基于 XML 的配置方式将进一步简化
