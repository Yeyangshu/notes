# refresh

```java
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 准备资源，验证必需的属性
        prepareRefresh();

        // 创建一个ConfigurableListableBeanFactory作为基本的IOC容器，下面的操作都基于这个beanFactory进行的
        // 这里beanFactory已经把xml中定义的BeanDefinition加载入IOC容器中了。
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

        //设置beanFactory的基本属性
        prepareBeanFactory(beanFactory);

        try {
            // 通知子类来配置beanFactory，这里是空实现，不深入分析了
            postProcessBeanFactory(beanFactory);

            // 调用所有的BeanFactoryPostProcessor
            invokeBeanFactoryPostProcessors(beanFactory);

            // 注册BeanPostProcessor，真正调用的时候是getBean
            registerBeanPostProcessors(beanFactory);

            // 初始化MessageSoruce，即不同语言的消息体，国际化处理
            initMessageSource();

            // 初始化应用消息广播器
            initApplicationEventMulticaster();

            // 通知子类刷新容器
            onRefresh();

            // 注册事件监听器
            registerListeners();

            // 初始化单例对象
            finishBeanFactoryInitialization(beanFactory);

            // 收尾
            finishRefresh();
        }
        //出现异常
        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
            }

            // 销毁bean
            destroyBeans();

            // 重置 'active' 标志.
            cancelRefresh(ex);

            throw ex;
        }
    }
}
```

# invokeBeanFactoryPostProcessors

## 1 BeanFactoryPostProcessor

Spring IOC 容器允许 BeanFactoryPostProcessor 在容器实际实例化任何其他的 bean 之前读取配置元数据，并有可能修改它。还能通过设置`order`属性来控制 BeanFactoryPostProcessor 的执行次序。

BeanFactoryPostProcessor 的作用域范围是容器级的。它只和你所使用的容器有关。如果你再容器中定义一个 BeanFactoryPostProcessor，它仅仅对此容器中的 bean 进行后置处理。



其实就是来定制和修改BeanFactory的内容，如覆盖或添加属性





BeanFactoryPostProcessor

```java
public interface BeanFactoryPostProcessor {

	/**
	 * 在标准初始化之后，修改应用程序上下文的内部bean工厂。所有bean定义都将被加载，但尚未实例化任何bean。 这甚至可以覆盖或添加属性，甚至可以用于初始化bean。
	 */
	void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;

}
```

子接口 BeanDefinitionRegistryPostProcessor

```java
public interface BeanDefinitionRegistryPostProcessor extends BeanFactoryPostProcessor {

	/**
	 * 标准初始化后，修改应用程序上下文的内部bean定义注册表。
	 * 所有常规bean定义都将被加载，但是尚未实例化任何bean。 这允许在下一个后处理阶段开始之前添加更多的bean定义。
	 */
	void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException;

}
```

## 2 BeanFactoryPostProcessor 的处理

BeanFactoryPostProcessor 的处理主要分为两种情况进行：

- 对 BeanDefinitionRegistry 类的特殊处理
- 对普通的 BeanFactoryPostProcessor 进行处理

### 2.1 对 BeanDefinitionRegistry 类的处理

#### 2.1.1 对于硬编码注册的后置处理器的处理

添加 beanFactoryPostProcessor：AbstractApplicationContext 类中的 addBeanFactoryPostProcessor 方法

```java
@Override
public void addBeanFactoryPostProcessor(BeanFactoryPostProcessor postProcessor) {
    Assert.notNull(postProcessor, "BeanFactoryPostProcessor must not be null");
    this.beanFactoryPostProcessors.add(postProcessor);
}
```

查找 beanFactoryPostProcessor：getBeanFactoryPostProcessors()

处理 beanFactoryPostProcessor：处理 BeanFactoryPostProcessor 时候首先会检测 beanFactoryPostProcessors 是否有数据。BeanDefinitionRegistryPostProcessor 继承自 BeanFactoryPostProcessor，不但有 BeanFactoryPostProcessor  的特性，同时还有自己定义的个性化方法，也需要在此调用。所以，这里需要从 beanFactoryPostProcessors 中挑出 BeanDefinitionRegistryPostProcessor  的后处理器，并进行 postProcessBeanDefinitionRegistry 方法的激活。

#### 2.1.2 记录后处理器

记录后处理器主要使用了三个 List：

- registryProcessors：记录通过硬编码方式注册的 BeanDefinitionRegistryPostProcessor 类型的处理器
- regularPostProcessors：记录通过硬编码方式注册的 BeanFactoryPostProcessor 类型的处理器
- currentRegistryProcessors：记录通过配置方式注册的 BeanDefinitionRegistryPostProcessor 类型的处理器

#### 2.1.3 调用 postProcessBeanFactory 方法

对以上所记录的 List 中 的后处理器进行统一调用 BeanFactoryPostProcessor 的 postProcessBeanFactory 方法。

#### 2.1.4 非 BeanDefinitionRegistryPostProcessor 的 postProcessBeanFactory  调用

对 beanFactoryPostProcessors 中非 BeanDefinitionRegistryPostProcessor 类型的后处理器进行统一的 BeanFactoryPostProcessor 的  postProcessBeanFactory 方法调用。

### 2.2 普通 beanFactory 处理

BeanDefinitionRegistryPostProcessor 只对 BeanDefinitionRegistry 类型的 ConfigurableListableBeanFactory 有效，所以如果判断所示的 beanFactory 并不是  BeanDefinitionRegistry ，那么便可以忽略 BeanDefinitionRegistryPostProcessor，而直接处理 BeanFactoryPostProcessor。



执行过程

```java
public static void invokeBeanFactoryPostProcessors(
			ConfigurableListableBeanFactory beanFactory, List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {

		// Invoke BeanDefinitionRegistryPostProcessors first, if any.
    	// 如果有的话，首先调用 BeanDefinitionRegistryPostProcessors
    
    	// 用于存放已处理过的Bean
		Set<String> processedBeans = new HashSet<>();

    	// 对 BeanDefinitionRegistry 类型的处理
		if (beanFactory instanceof BeanDefinitionRegistry) {
			BeanDefinitionRegistry registry = (BeanDefinitionRegistry) beanFactory;
			List<BeanFactoryPostProcessor> regularPostProcessors = new ArrayList<>();
			List<BeanDefinitionRegistryPostProcessor> registryProcessors = new ArrayList<>();

            // 遍历硬编码注册的后置处理器
			for (BeanFactoryPostProcessor postProcessor : beanFactoryPostProcessors) {
				if (postProcessor instanceof BeanDefinitionRegistryPostProcessor) {
					BeanDefinitionRegistryPostProcessor registryProcessor =
							(BeanDefinitionRegistryPostProcessor) postProcessor;
                    // 对于 BeanDefinitionRegistryPostProcessor 类型，在 BeanFactoryPostProcessor 的基础上还有自己定义的方法，需要先调用
					registryProcessor.postProcessBeanDefinitionRegistry(registry);
                    // 添加到 registryProcessors 列表中
					registryProcessors.add(registryProcessor);
				}
				else {
                    // 记录常规的 BeanFactoryPostProcessor
					regularPostProcessors.add(postProcessor);
				}
			}

			// Do not initialize FactoryBeans here: We need to leave all regular beans
			// uninitialized to let the bean factory post-processors apply to them!
			// Separate between BeanDefinitionRegistryPostProcessors that implement
			// PriorityOrdered, Ordered, and the rest.
            
            // 配置注册的后置处理器列表
			List<BeanDefinitionRegistryPostProcessor> currentRegistryProcessors = new ArrayList<>();

			// First, invoke the BeanDefinitionRegistryPostProcessors that implement PriorityOrdered.
			String[] postProcessorNames =
					beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
            // 排序
			sortPostProcessors(currentRegistryProcessors, beanFactory);
            // 添加到 BeanDefinitionRegistryPostProcessor 的 List
			registryProcessors.addAll(currentRegistryProcessors);
            // 激活 postProcessBeanDefinitionRegistry
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// Next, invoke the BeanDefinitionRegistryPostProcessors that implement Ordered.
			postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
			for (String ppName : postProcessorNames) {
				if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
					currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
					processedBeans.add(ppName);
				}
			}
			sortPostProcessors(currentRegistryProcessors, beanFactory);
			registryProcessors.addAll(currentRegistryProcessors);
			invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
			currentRegistryProcessors.clear();

			// Finally, invoke all other BeanDefinitionRegistryPostProcessors until no further ones appear.
			boolean reiterate = true;
			while (reiterate) {
				reiterate = false;
				postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
				for (String ppName : postProcessorNames) {
					if (!processedBeans.contains(ppName)) {
						currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
						processedBeans.add(ppName);
						reiterate = true;
					}
				}
				sortPostProcessors(currentRegistryProcessors, beanFactory);
				registryProcessors.addAll(currentRegistryProcessors);
				invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
				currentRegistryProcessors.clear();
			}

			// Now, invoke the postProcessBeanFactory callback of all processors handled so far.
			invokeBeanFactoryPostProcessors(registryProcessors, beanFactory);
			invokeBeanFactoryPostProcessors(regularPostProcessors, beanFactory);
		}

		else {
			// Invoke factory processors registered with the context instance.
			invokeBeanFactoryPostProcessors(beanFactoryPostProcessors, beanFactory);
		}

		// Do not initialize FactoryBeans here: We need to leave all regular beans
		// uninitialized to let the bean factory post-processors apply to them!
		String[] postProcessorNames =
				beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);

		// Separate between BeanFactoryPostProcessors that implement PriorityOrdered,
		// Ordered, and the rest.
		List<BeanFactoryPostProcessor> priorityOrderedPostProcessors = new ArrayList<>();
		List<String> orderedPostProcessorNames = new ArrayList<>();
		List<String> nonOrderedPostProcessorNames = new ArrayList<>();
		for (String ppName : postProcessorNames) {
			if (processedBeans.contains(ppName)) {
				// skip - already processed in first phase above
			}
			else if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
				priorityOrderedPostProcessors.add(beanFactory.getBean(ppName, BeanFactoryPostProcessor.class));
			}
			else if (beanFactory.isTypeMatch(ppName, Ordered.class)) {
				orderedPostProcessorNames.add(ppName);
			}
			else {
				nonOrderedPostProcessorNames.add(ppName);
			}
		}

		// First, invoke the BeanFactoryPostProcessors that implement PriorityOrdered.
		sortPostProcessors(priorityOrderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(priorityOrderedPostProcessors, beanFactory);

		// Next, invoke the BeanFactoryPostProcessors that implement Ordered.
		List<BeanFactoryPostProcessor> orderedPostProcessors = new ArrayList<>(orderedPostProcessorNames.size());
		for (String postProcessorName : orderedPostProcessorNames) {
			orderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		sortPostProcessors(orderedPostProcessors, beanFactory);
		invokeBeanFactoryPostProcessors(orderedPostProcessors, beanFactory);

		// Finally, invoke all other BeanFactoryPostProcessors.
		List<BeanFactoryPostProcessor> nonOrderedPostProcessors = new ArrayList<>(nonOrderedPostProcessorNames.size());
		for (String postProcessorName : nonOrderedPostProcessorNames) {
			nonOrderedPostProcessors.add(beanFactory.getBean(postProcessorName, BeanFactoryPostProcessor.class));
		}
		invokeBeanFactoryPostProcessors(nonOrderedPostProcessors, beanFactory);

		// Clear cached merged bean definitions since the post-processors might have
		// modified the original metadata, e.g. replacing placeholders in values...
		beanFactory.clearMetadataCache();
	}
```

这个要看ConfigurationClassPostProcessor



## 自定义BeanFactoryPostProcessor

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
	@Override
	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
		System.out.println("-----------");
	}
}
```













测试代码

```java
public class Tank implements Movable {

    /**
     * 模拟坦克移动了一段儿时间
     */
    @Override
    public void move() {
        System.out.println("Tank moving claclacla...");
        try {
            Thread.sleep(new Random().nextInt(10000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Tank tank = new Tank();

        System.getProperties().put("jdk.proxy.ProxyGenerator.saveGeneratedFiles", "true");

        /**
         * Proxy.newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)
         * 返回指定接口的代理类的实例，该实例将方法调用分派到指定的调用处理程序。
         *
         * 参数：
         * loader –定义代理类的类加载器
         * interfaces –代理类要实现的接口的列表
         * h –将方法调用分派到的调用处理程序
         */
        Movable m = (Movable) Proxy.newProxyInstance(Tank.class.getClassLoader(),
                new Class[]{Movable.class},
                new TimeProxy(tank)
        );

        m.move();

    }
}

class TimeProxy implements InvocationHandler {
    Movable m;

    public TimeProxy(Movable m) {
        this.m = m;
    }

    public void before() {
        System.out.println("method start..");
    }

    public void after() {
        System.out.println("method stop..");
    }

    /**
     * 处理代理实例上的方法调用并返回结果。 在与其关联的代理实例上调用方法时，将在调用处理程序上调用该方法。
     *
     * 参数：
     * proxy –在其上调用方法的代理实例，本次是 m
     * method –与在代理实例上调用的接口方法相对应的Method实例，本次是 move()。 Method对象的声明类将是在其中声明该Method的接口，它可以是代理类通过其继承该方法的代理接口的超接口。
     * args –包含在代理实例的方法调用中传递的参数值的对象数组；如果接口方法不带参数，则为null 。 基本类型的参数包装在适当的基本包装器类的实例中，例如java.lang.Integer或java.lang.Boolean 。
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 类里面的所有方法：Arrays.stream(proxy.getClass().getMethods()).map(Method::getName).forEach(System.out::println);
        before();
        // 传给哪个对象引用就调用对应的 move() 方法，相当于调用了 tank.move() 方法
        Object o = method.invoke(m, args);
        after();
        return o;
    }

}

interface Movable {
    void move();
}
```

源码：

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;


final class $Proxy0 extends Proxy implements Movable {
    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;

    /**
     * 构造方法
     *
     * @param var1 InvocationHandler 的对象，本次是 TimeProxy 对象
     */
    public $Proxy0(InvocationHandler var1) throws {
        super(var1);
    }

    public final boolean equals(Object var1) throws {
        try {
            return (Boolean) super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    /**
     * 生成了 move() 方法
     * 实际调用的是 super.h.invoke，也就是 TimeProxy 的 invoke() 方法
     * 第一个参数是 this：生成的代理对象，代码里的 m
     */
    public final void move() throws {
        try {
            super.h.invoke(this, m3, (Object[]) null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws {
        try {
            return (String) super.h.invoke(this, m2, (Object[]) null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws {
        try {
            return (Integer) super.h.invoke(this, m0, (Object[]) null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.mashibing.dp.proxy.v10.Movable").getMethod("move");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```







## ConfigurationClassPostProcessor

首先会执行 postProcessBeanDefinitionRegistry



```java
/**
 * 构建和验证一个类是否被@Configuration修饰，并做相关的解析工作
 *
 * 如果你对此方法了解清楚了，那么springboot的自动装配原理就清楚了
 *
 * Build and validate a configuration model based on the registry of
 * {@link Configuration} classes.
 */
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    // 创建存放BeanDefinitionHolder的对象集合
    List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
    // 当前registry就是DefaultListableBeanFactory，获取所有已经注册的BeanDefinition的beanName
    String[] candidateNames = registry.getBeanDefinitionNames();

    // 遍历所有要处理的beanDefinition的名称，筛选对应的beanDefinition（被注解修饰的）
    for (String beanName : candidateNames) {
        // 获取指定名称的BeanDefinition对象
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        // 如果beanDefinition中的configurationClass属性不等于空，那么意味着已经处理过，输出日志信息
        if (beanDef.getAttribute(ConfigurationClassUtils.CONFIGURATION_CLASS_ATTRIBUTE) != null) {
            if (logger.isDebugEnabled()) {
                logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
            }
        }
        // 判断当前BeanDefinition是否是一个配置类，并为BeanDefinition设置属性为lite或者full，此处设置属性值是为了后续进行调用
        // 如果Configuration配置proxyBeanMethods代理为true则为full
        // 如果加了@Bean、@Component、@ComponentScan、@Import、@ImportResource注解，则设置为lite
        // 如果配置类上被@Order注解标注，则设置BeanDefinition的order属性值
        else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
            // 添加到对应的集合对象中
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

    // Return immediately if no @Configuration classes were found
    // 如果没有发现任何配置类，则直接返回
    if (configCandidates.isEmpty()) {
        return;
    }

    // Sort by previously determined @Order value, if applicable
    // 如果适用，则按照先前确定的@Order的值排序
    configCandidates.sort((bd1, bd2) -> {
        int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
        int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
        return Integer.compare(i1, i2);
    });

    // Detect any custom bean name generation strategy supplied through the enclosing application context
    // 判断当前类型是否是SingletonBeanRegistry类型
    SingletonBeanRegistry sbr = null;
    if (registry instanceof SingletonBeanRegistry) {
        // 类型的强制转换
        sbr = (SingletonBeanRegistry) registry;
        // 判断是否有自定义的beanName生成器
        if (!this.localBeanNameGeneratorSet) {
            // 获取自定义的beanName生成器
            BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(
                AnnotationConfigUtils.CONFIGURATION_BEAN_NAME_GENERATOR);
            // 如果有自定义的命名生成策略
            if (generator != null) {
                //设置组件扫描的beanName生成策略
                this.componentScanBeanNameGenerator = generator;
                // 设置import bean name生成策略
                this.importBeanNameGenerator = generator;
            }
        }
    }

    // 如果环境对象等于空，那么就重新创建新的环境对象
    if (this.environment == null) {
        this.environment = new StandardEnvironment();
    }

    // Parse each @Configuration class
    // 实例化ConfigurationClassParser类，并初始化相关的参数，完成配置类的解析工作
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);

    // 创建两个集合对象，
    // 存放相关的BeanDefinitionHolder对象
    Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
    // 存放扫描包下的所有bean
    Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
    do {
        // 解析带有@Controller、@Import、@ImportResource、@ComponentScan、@ComponentScans、@Bean的BeanDefinition
        parser.parse(candidates);
        // 将解析完的Configuration配置类进行校验，1、配置类不能是final，2、@Bean修饰的方法必须可以重写以支持CGLIB
        parser.validate();

        // 获取所有的bean,包括扫描的bean对象，@Import导入的bean对象
        Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
        // 清除掉已经解析处理过的配置类
        configClasses.removeAll(alreadyParsed);

        // Read the model and create bean definitions based on its content
        // 判断读取器是否为空，如果为空的话，就创建完全填充好的ConfigurationClass实例的读取器
        if (this.reader == null) {
            this.reader = new ConfigurationClassBeanDefinitionReader(
                registry, this.sourceExtractor, this.resourceLoader, this.environment,
                this.importBeanNameGenerator, parser.getImportRegistry());
        }
        // 核心方法，将完全填充好的ConfigurationClass实例转化为BeanDefinition注册入IOC容器
        this.reader.loadBeanDefinitions(configClasses);
        // 添加到已经处理的集合中
        alreadyParsed.addAll(configClasses);

        candidates.clear();
        // 这里判断registry.getBeanDefinitionCount() > candidateNames.length的目的是为了知道reader.loadBeanDefinitions(configClasses)这一步有没有向BeanDefinitionMap中添加新的BeanDefinition
        // 实际上就是看配置类(例如AppConfig类会向BeanDefinitionMap中添加bean)
        // 如果有，registry.getBeanDefinitionCount()就会大于candidateNames.length
        // 这样就需要再次遍历新加入的BeanDefinition，并判断这些bean是否已经被解析过了，如果未解析，需要重新进行解析
        // 这里的AppConfig类向容器中添加的bean，实际上在parser.parse()这一步已经全部被解析了
        if (registry.getBeanDefinitionCount() > candidateNames.length) {
            String[] newCandidateNames = registry.getBeanDefinitionNames();
            Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
            Set<String> alreadyParsedClasses = new HashSet<>();
            for (ConfigurationClass configurationClass : alreadyParsed) {
                alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
            }
            // 如果有未解析的类，则将其添加到candidates中，这样candidates不为空，就会进入到下一次的while的循环中
            for (String candidateName : newCandidateNames) {
                if (!oldCandidateNames.contains(candidateName)) {
                    BeanDefinition bd = registry.getBeanDefinition(candidateName);
                    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                        !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                        candidates.add(new BeanDefinitionHolder(bd, candidateName));
                    }
                }
            }
            candidateNames = newCandidateNames;
        }
    }
    while (!candidates.isEmpty());

    // Register the ImportRegistry as a bean in order to support ImportAware @Configuration classes
    if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
        sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
    }

    if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
        // Clear cache in externally provided MetadataReaderFactory; this is a no-op
        // for a shared cache since it'll be cleared by the ApplicationContext.
        ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
    }
}
```


