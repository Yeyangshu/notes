# 源码需要掌握的点

**源码研究目的**

1. **面试**
2. **二次开发**
3. **设计模式思想**



1. 学习反射

2. 设计模式

   - 观察者模式
   - 适配器模式： loadBeanDefinitions

3. 需要掌握的 BeanFactory 接口

   - BeanFactory 

   - ListableBeanFactory

   - ConfigurableBeanFactory

   - DefaultListableBeanFactory

![image-20201125231143870](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201125231143870.png)

4. refresh() 方法注释

   ```java
   @Override
   	public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// Prepare this context for refreshing.
               // 准备资源，验证必需的属性，容器刷新前的准备工作
               // 1 设置容器的启动时间
               // 2 设置活跃状态为true
               // 3 设置关闭状态为false
               // 4 获取Enviroment对象，并加载当前系统的属性值到Enviroment对象中
               // 5 准备监听器和事件的集合队列，默认为空的集合
   			prepareRefresh();
   
   			// Tell the subclass to refresh the internal bean factory.
               // 创建容器对象：DefaultListableBeanFactory，作为基本的IOC容器，下面的操作都基于这个beanFactory进行的
               // 加载xml配置文件的属性值到当前的工厂中，最重要的就是BeanDefinition
   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
   			// Prepare the bean factory for use in this context.
               // 设置beanFactory的基本属性,初始化
   			prepareBeanFactory(beanFactory);
   
   			try {
   				// Allows post-processing of the bean factory in context subclasses.
                   // 通知子类来配置beanFactory，这里是空实现，
   				postProcessBeanFactory(beanFactory);
   
   				// Invoke factory processors registered as beans in the context.
                   // 调用所有的BeanFactoryPostProcessor
   				invokeBeanFactoryPostProcessors(beanFactory);
   
   				// Register bean processors that intercept bean creation.
                   // 注册BeanPostProcessor
   				registerBeanPostProcessors(beanFactory);
   
   				// Initialize message source for this context.
                   // 初始化MessageSoruce
   				initMessageSource();
   
   				// Initialize event multicaster for this context.
                   // 初始化事件分发器
   				initApplicationEventMulticaster();
   
   				// Initialize other special beans in specific context subclasses.
                   // 通知子类刷新容器
   				onRefresh();
   
   				// Check for listener beans and register them.
                   // 注册事件监听器
   				registerListeners();
   
   				// Instantiate all remaining (non-lazy-init) singletons.
                   // 初始化单例对象
   				finishBeanFactoryInitialization(beanFactory);
   
   				// Last step: publish corresponding event.
   				finishRefresh();
   			}
   
   			catch (BeansException ex) {
   				if (logger.isWarnEnabled()) {
   					logger.warn("Exception encountered during context initialization - " +
   							"cancelling refresh attempt: " + ex);
   				}
   
   				// Destroy already created singletons to avoid dangling resources.
   				destroyBeans();
   
   				// Reset 'active' flag.
   				cancelRefresh(ex);
   
   				// Propagate exception to caller.
   				throw ex;
   			}
   
   			finally {
   				// Reset common introspection caches in Spring's core, since we
   				// might not ever need metadata for singleton beans anymore...
   				resetCommonCaches();
   			}
   		}
   	}
   ```

5. BeanDefinitions关系图

   ![](https://yeyangshu-picgo.oss-cn-shanghai.aliyuncs.com/img/image-20201201000035769.png)

6. 记住这两个 beanDefinitionMap 和 beanDefinitionNames

   ```java
   this.beanDefinitionMap.put(beanName, beanDefinition);
   this.beanDefinitionNames.add(beanName);
   ```

   

