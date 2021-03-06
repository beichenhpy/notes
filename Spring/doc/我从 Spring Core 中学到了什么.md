# 代码书写

让别人看代码，像看报纸一样，流畅一些。

## 类属性、方法顺序

+ public 方法早出现（可选，最好是同一访问权限的类变量和方法就近原则）。
+ 应用范围比较广的类属性，标明在哪使用。
+ static 变量和 static 代码块早出现于普通变量。
+ 代码块/方法和变量之间隔两行书写，方法和方法，属性和属性隔一行。
+ 不太复杂的类属性，单行注释即可。
+ 可能并且允许为空的对象，用 @Nullable 标记。
+ 构造方法优先于普通方法出现在变量下面。
+ 无参构造方法早出现于有参构造法。
+ 非 static 修饰的变量，不应直接引用。
+ 在不确定对象是否为空的情况下，尽量兜底。
+ final 并且未被 static 修饰的变量，可以不进行检查，通过方法调用，而不是直接使用变量。
+ 方法就近原则，方法内调用当前类其他方法时，应当在调用下面，越早调用，越早出现。
+ 方法内写代码时，一段语义可以适当与另一段语义空一行。
+ 方法与类之间留一行空格

**AbstractApplicationContext**

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {

    /**
 	 * public 方法早出现（可选，最好是同一访问权限的类变量和方法就近原则）。
 	 * 应用范围比较广的类属性，标明在哪使用。
 	 * Name of the MessageSource bean in the factory.
 	 * If none is supplied, message resolution is delegated to the parent.
 	 * @see MessageSource
 	 */
    public static final String MESSAGE_SOURCE_BEAN_NAME = "messageSource";
    
    
    //static 变量和 static 代码块早出现于普通变量。
    static {
       ContextClosedEvent.class.getName();
    }
    
    
    // 代码块/方法和变量之间隔两行书写，方法和方法，属性和属性隔一行。
    // 不太复杂的类属性，单行注释即可。
    /** Logger used by this class. Available to subclasses. */
	protected final Log logger = LogFactory.getLog(getClass());
    
    /** Unique id for this context, if any. */
	private String id = ObjectUtils.identityToString(this);
    
    // 可能并且允许为空的对象，用 @Nullable 标记。
    @Nullable
	private ConfigurableEnvironment environment;

    
    // 构造方法优先于普通方法出现在变量下面。
	public AbstractApplicationContext() {
		this.resourcePatternResolver = getResourcePatternResolver();
	}
    
    // 无参构造方法早出现于有参构造法。
    public AbstractApplicationContext(@Nullable ApplicationContext parent) {
		this();
		setParent(parent);
	}

    /**
	 * 非 static 修饰的变量，不应直接引用。
	 * 在不确定对象是否为空的情况下，尽量兜底
	 */
	@Override
	public ConfigurableEnvironment getEnvironment() {
		if (this.environment == null) {
			this.environment = createEnvironment();
		}
		return this.environment;
	}
	
    // final 并且未被 static 修饰的变量，可以不进行检查，通过方法调用，而不是直接使用变量。
	public Collection<ApplicationListener<?>> getApplicationListeners() {
		return this.applicationListeners;
	}
    
    // 方法就近原则，方法内调用当前类其他方法时，应当在调用下面，越早调用，越早出现。
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();
            // 省略一大段代码
		}
	}

	protected void prepareRefresh() {
        // 方法内写代码时，一段语义可以适当与另一段语义空一行。
		// Switch to active.
		this.startupDate = System.currentTimeMillis();
		this.closed.set(false);
		this.active.set(true);

		if (logger.isDebugEnabled()) {
			if (logger.isTraceEnabled()) {
				logger.trace("Refreshing " + this);
			}
			else {
				logger.debug("Refreshing " + getDisplayName());
			}
		}

		// Initialize any placeholder property sources in the context environment.
		initPropertySources();

		// Validate that all properties marked as required are resolvable:
		// see ConfigurablePropertyResolver#setRequiredProperties
		getEnvironment().validateRequiredProperties();

		// Store pre-refresh ApplicationListeners...
		if (this.earlyApplicationListeners == null) {
			this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
		}
		else {
			// Reset local application listeners to pre-refresh state.
			this.applicationListeners.clear();
			this.applicationListeners.addAll(this.earlyApplicationListeners);
		}

		// Allow for the collection of early ApplicationEvents,
		// to be published once the multicaster is available...
		this.earlyApplicationEvents = new LinkedHashSet<>();
	}        // 方法与类之间留一行空格
    
}
```

## 将复杂的代码块拆分成多个小的私有方法

具体见各个版本的方法。并且私有方法，不要有注释。

例如 **ClassPathScanningCandidateComponentProvider#findCandidateComponents** 在 3.0 版本，这个方法又臭又长。

```java
/**
 * Scan the class path for candidate components.
 * @param basePackage the package to check for annotated classes
 * @return a corresponding Set of autodetected bean definitions
 */
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
   if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
      return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
   }
   else {
      return scanCandidateComponents(basePackage);
   }
}
```

# 培养抽象思维

Resource

AutowireCandidateResolver

BeanDefinitionRegistry

BeanNameGenerator

MergedBeanDefinitionPostProcessor（Bean 的生命周期有一个 BeanDefinition merge 的过程，调用的这个）

Aware（意识，一般是用来注入各个其子类的，例如 ApplicationContextAware，ApplicationEventPublisherAware，后者其实一般还是 ApplicationContext，BeanFactoryAware）

DisposableBean

FactoryBean

HierarchicalBeanFactory（层次性 BeanFactory，标记，然后可以获得具体的 parentBeanFactory）

InitializingBean

ListableBeanFactory（列表式的 BeanFactory）

NamedBean（自定义 Bean 名称）

ObjectFactory（对象工厂，其子类，DefaultListableBeanFactory）

ObjectProvider（对象提供者，其子类，DefaultListableBeanFactory）

SmartInitializingSingleton（这个在 DefaultListableBeanFactory 会有在获取 Bean 前进行一波操作的 ）

BeanMetadataElement（其子类接口 BeanDefinition）

BeanWrapper（增强 Bean 用的包装类）

PropertyValues（典型的子类实现 MutablePropertyValues，其实就是套了多个 PropertyValue，多用于获取配置信息）

Environment（所有环境配置的抽象，会覆盖掉）

Cache（缓存抽象）

CacheManager（管理缓存）

ApplicationEventMulticaster（事件广播器）

EventListenerFactory（事件监听工厂）
LocaleContext（国际化支持上下文抽象）
ApplicationContextAware（获取 ApplicationContext 引用）

ApplicationEventPublisher（应用事件发布者，典型的实现为 AbstractApplicationContext）

所以说 ApplicationContext 即是 ResourceLoder 也是事件发布者

Lifecycle （一般是 ApplicationContext 的生命周期接口）

MessageSource（消息源，一般也是 ApplicationContext）

SmartLifecycle

Formatter（日期，金额等等格式化的接口）

JmxAttributeSource（JMX 想关的）

RmiInvocationHandler（配置 Rmi 相关的）

RemoteInvocationExecutor，

RemoteInvocationFactory

SchedulingTaskExecutor（定时任务相关）

TaskScheduler 

Trigger

TriggerContext

HierarchicalThemeSource（主题相关的，和 Spring MVC 相关，当然你做 GUI 项目，我也不拦着）

ThemeSource，

Theme

Model（Spring validation 相关）

Profiles

PropertyResolver（Environment 相关）

PropertySources

WritableResource（继承自 Resource，主要是就是表明这是个可写的文件）

MetadataReader（ASM 抽象的类的属性）

BeanDefinition

BeanFactory 

ApplicationContext（增强版 BeanFactory）

BeanPostProcessor

BeanFactoryProcessor

TransactionDefinition（事务抽象传播以及隔离级别，在这定义）

AopProxyFactory（Aop 方面的接口）

Advice（AfterAdvice，AfterReturningAdvice，BeforeAdvice 各种时候织入的接口）

Advisor（例如 DefaultPointcutAdvisor）

Scope（定义作用域的接口）

JDK 自带的 EventListener

大家默认继承这个，算是个遵守约定。







```
/**
 * Simple facade for accessing class metadata,
 * as read by an ASM {@link org.springframework.asm.ClassReader}.
 *
 * @author Juergen Hoeller
 * @since 2.5
 */
```