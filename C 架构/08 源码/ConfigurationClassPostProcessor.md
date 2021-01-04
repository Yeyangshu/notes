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

解析