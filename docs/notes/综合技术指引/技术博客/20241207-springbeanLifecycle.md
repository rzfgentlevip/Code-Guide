---
title: SpringBean生命周期
icon: material-icon-theme:screwdriver
order: 5
author: bugcode
date: 2024-12-7
category:
  - SPRING
tag:
  - 面试
  - spring
sticky: false
star: true
footer: 分布式
copyright: bugcode
createTime: 2025/09/04 10:31:15
permalink: /compre-guide/technical-blog/springbeanLifecycle/
---

# 1、SpringBean生命周期

## 什么是Spring Bean生命周期

普通的java对象，当 new 的时候创建对象然后将对象放到内存中，然后该对象就能够使用了。一旦该对象不再被使用，则由 Java 自动进行垃圾回收。

spring容器中的bean对象，bean 和普通的 Java 对象没啥大的区别，只不过 Spring 不再自己去 new 对象了，而是由 IoC 容器去帮助我们实例化对象并且管理它，我们需要哪个对象，去问 IoC 容器要即可。IoC 其实就是解决对象之间的耦合问题，Spring Bean 的生命周期完全由容器控制。

因此一个springIOC容器至少有以下功能：

- 配置解析
- 对象创建
- 对象生命周期管理

因此从启动spring容器调用方法栈来看，spring容器管理的bean对象主要有5个阶段：

1. BeanDefinition定义加载和注册
2. Bean对象实例化
3. Bean对象属性赋值
4. Bean对象初始化
5. Bean对象销毁

springBean对象的作用域：

- singleton : 唯一 bean 实例，Spring 中的 bean 默认都是单例的。
- prototype : 每次请求都会创建一个新的 bean 实例。
- request : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- session : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
- global-session： 全局 session 作用域，仅仅在基于 Portlet 的 web 应用中才有意义，Spring5 已经没有了。Portlet 是能够生成语义代码(例如：HTML)片段的小型 Java Web 插件。它们基于 portlet 容器，可以像 servlet 一样处理 HTTP 请求。但是，与 servlet 不同，每个 portlet 都有不同的会话。

## 生命周期方法分类

Bean自身的方法：这个包括了Bean本身调用的方法(getter/setter等方法)和通过配置文件中bean的init-method和destroy-method指定的方法

Bean级生命周期接口方法：这个包括了BeanNameAware、BeanFactoryAware、InitializingBean(初始化)和DiposableBean(销毁)这些接口的方法

容器级生命周期接口方法：这个包括了InstantiationAwareBeanPostProcessor(创建bean对象前后执行) 和 BeanPostProcessor (bean前置和后置处理器)这两个接口实现，一般称它们的实现类为“后处理器”。

工厂后处理器接口方法：这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor,  CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

## 源码解析

### 注册阶段

注册阶段的主要任务是通过各种BeanDefinitionReader读取各种配置来源信息(比如读取xml文件、注解等)，并将其转化为BeanDefintion的过程。

这里要理解一下BeanDefinition的作用。众所周知，Spring提供了多种多样的注册Bean的方法，BeanDefinition的作用就是去定义并描述一个Spring Bean，方便后续解析实例化等操作。

ApplicationContext#register()方法完成了对象注册阶段，其最终是调用的DefaultListableBeanFactory#registerBeanDefinition() 完成的BeanDefinition注册，这里的“注册”意思是将配置信息转化为BeanDefinition并放到合适的容器中，我们可以看以下代码：

```java
public class DefaultListableBeanFactory extends AbstractAutowireCapableBeanFactory
		implements ConfigurableListableBeanFactory, BeanDefinitionRegistry, Serializable {

	/** Map of bean definition objects, keyed by bean name. */
	private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap<>(256);

	/** List of bean definition names, in registration order. */
	private volatile List<String> beanDefinitionNames = new ArrayList<>(256);

	//省略部分代码......
	@Override
	public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {
				//省略了一些判断代码......
				this.beanDefinitionMap.put(beanName, beanDefinition);
				this.beanDefinitionNames.add(beanName);
				removeManualSingletonName(beanName);
	}
}

```

其中，beanDefinitionMap的数据结构是ConcurrentHashMap，因此不能保证顺序，为了记录注册的顺序，这里使用了ArrayList类型的beanDefinitionNames用来记录注册顺序,key保存的是对象的类型，val保存的是对象BeanDefinition信息。

### 合并阶段

经过了注册阶段，Spring的BeanDefinition容器中已经有了部分BeanDefinition信息(可能还存在通过aware接口或者postProcessor接口注册进来的beanDefinition)，下面分为两种情况：

1. 对于设置了非懒加载属性的BeanDefinition，在容器启动时(ApplicationContext#refresh())时会最终调用BeanFactory#getBean()方法进行实例化
2. 对于懒加载(isLazyInit)的BeanDefinition，则需要在用到的时候调用BeanFactory#getBean()方法进行实例化。

事实上，无论哪种情况，Spring最终都会调用BeanFactory#getBean()方法进行实例化。在getBean()方法中会有一个合并阶段：

```java
public abstract class AbstractBeanFactory extends FactoryBeanRegistrySupport implements ConfigurableBeanFactory {
	protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
			@Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
		//......
		final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
		//......
	}
}
```

AbstractBeanFactory#getMergedLocalBeanDefinition()方法完成了BeanDefinition的合并，这里的“合并”的意思是，部分BeanDefinition可能不是RootBeanDefinition(没有parent)，而是部分属性需要继承其他BeanDefinition，比如xml配置中的parent属性，这就需要进行一次合并，最终产出RootBeanDefinition。

RootBeanDefinition的parent设置时候会有一个判断，可以看出来，RootBeanDefinition最典型的特点就是没有parent reference：

```java
public class RootBeanDefinition extends AbstractBeanDefinition {
	//......
	@Override
	public void setParentName(@Nullable String parentName) {
		if (parentName != null) {
			throw new IllegalArgumentException("Root bean cannot be changed into a child bean with parent reference");
		}
	}
	//......
}
```

### 实例化阶段

到了实例化阶段，Spring将转化BeanDefinition中BeanDefinition为实例Bean(放在包装类BeanWrapper中)。

#### 通过ClassLoader赋值BeanDefinition的beanClass为Class对象

我们首先关注到AbstractAutowireCapableBeanFactory#createBean()方法，AbstractAutowireCapableBeanFactory是DefaultListableBeanFactory的父类，在这个方法中有个过程：

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	@Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		if (logger.isTraceEnabled()) {
			logger.trace("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

		//......

		// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
		Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
		if (bean != null) {
				return bean;
		}
		//......

		// Make sure bean class is actually resolved at this point, and
		// clone the bean definition in case of a dynamically resolved Class
		// which cannot be stored in the shared merged bean definition.
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}
		//......	
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		//......	
	}
	//......
}
```

可以看到#resolveBeanClass()方法，我们知道，比如通过xml的方式定义的BeanDefinition的beanClass是个字符串，因此这里需要通过这个方法加载Class对象并赋值回beanClass这个属性中。

当然这个细节只是顺便提一下，并不是Bean生命周期中需要太关注的一环。

#### 实例化前阶段resolveBeforeInstantiation

从上文的代码中我们可以看这个方法AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation()，这就是实例化前阶段，主要是处理注册到容器中的InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation() ，如果有返回值则将直接用返回值作为实例好的bean进行返回。

可以看到具体代码：

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	@Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// Make sure bean class is actually resolved at this point.
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
	//......
}
```

#### 实例化阶段createBeanInstance

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		//......
		populateBean(beanName, mbd, instanceWrapper);
		exposedObject = initializeBean(beanName, exposedObject, mbd);
		//......

		return exposedObject;
	}
	//......
}
```

在实例化阶段中， AbstractAutowireCapableBeanFactory#createBeanInstance()方法完成了Bean的创建，并将其放在包装类BeanWrapper中。

#### 实例化后阶段populateBean

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
		if (bw == null) {
			if (mbd.hasPropertyValues()) {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
			}
			else {
				// Skip property population phase for null instance.
				return;
			}
		}

		// Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
		// state of the bean before properties are set. This can be used, for example,
		// to support styles of field injection.
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
						return;
					}
				}
			}
		}

		PropertyValues pvs = (mbd.hasPropertyValues() ? mbd.getPropertyValues() : null);

		int resolvedAutowireMode = mbd.getResolvedAutowireMode();
		if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
			MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
			// Add property values based on autowire by name if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
				autowireByName(beanName, mbd, bw, newPvs);
			}
			// Add property values based on autowire by type if applicable.
			if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
				autowireByType(beanName, mbd, bw, newPvs);
			}
			pvs = newPvs;
		}

		boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
		boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);

		PropertyDescriptor[] filteredPds = null;
		if (hasInstAwareBpps) {
			if (pvs == null) {
				pvs = mbd.getPropertyValues();
			}
			for (BeanPostProcessor bp : getBeanPostProcessors()) {
				if (bp instanceof InstantiationAwareBeanPostProcessor) {
					InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
					PropertyValues pvsToUse = ibp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
					if (pvsToUse == null) {
						if (filteredPds == null) {
							filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
						}
						pvsToUse = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
						if (pvsToUse == null) {
							return;
						}
					}
					pvs = pvsToUse;
				}
			}
		}
		if (needsDepCheck) {
			if (filteredPds == null) {
				filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
			}
			checkDependencies(beanName, mbd, filteredPds, pvs);
		}

		if (pvs != null) {
			applyPropertyValues(beanName, mbd, bw, pvs);
		}
	}
	//......
}

```

简单总结，实例化后阶段#populateBean这个方法主要用来进行属性赋值(包括依赖注入)，分为以下三个阶段：

1. InstantiationAwareBeanPostProcessor#postProcessAfterInstantiation()，这个方法决定是否进行属性赋值(返回boolean值)
2. InstantiationAwareBeanPostProcessor#postProcessPropertyValues() ，这个方法可以改变具体属性的值。
3. applyPropertyValues()，进行属性赋值(包括依赖注入)。

#### 初始化阶段

初始化阶段的主要工作是在返回bean之前做一些处理，主要由AbstractAutowireCapableBeanFactory#initializeBean进行实现：

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
	//......
}
```

#### Bean Aware接口回调阶段

Spring提供了很多的Aware接口，这些接口都用于“赋予实现类感知xxx的能力”。比如用途最广泛的ApplicationContextAware接口，就是让实现类拥有了感知到applicationContext的能力，并能直接与applicationContext进行交互。

按照顺序，这些Aware接口有以下顺序：

![image-20240801114400566](https://vscodepic.oss-cn-beijing.aliyuncs.com/blog/202408011144819.png)

Aware接口回调主要依靠以下两个方法实现

1. AbstractAutowireCapableBeanFactory#invokeAwareMethods() ，主要处理列表前三种Aware接口
2. AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization() ，在ApplicationContext容器启动的时候会将除前3种Aware接口外的接口实现转化为BeanPostProcessor，进而在初始化阶段的时候进行调用，在代码中(上文中initializeBean方法)的调用顺序也是符合我们上述的顺序的。

#### 初始化前阶段applyBeanPostProcessorsBeforeInitialization

初始化前阶段主要通过 AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInitialization()方法进行实现，主要是处理 BeanPostProcessor#postProcessBeforeInitialization()，当然这里部分BeanPostProcessor也承担了Aware接口的回调任务，这些BeanPostProcessor是由ApplicationContext容器启动的时候注入的：

```java
public abstract class AbstractApplicationContext extends DefaultResourceLoader
		implements ConfigurableApplicationContext {
	//......
	protected void prepareBeanFactory(ConfigurableListableBeanFactory beanFactory) {
			//......
	
			// Configure the bean factory with context callbacks.
			beanFactory.addBeanPostProcessor(new ApplicationContextAwareProcessor(this));
	
			//......
	//......
}
```

这个阶段除此以外也承担了一些其他的功能，比如处理@PostConstruct注解等，Spring这种类似的拓展都是基于postProcessor去做的。

#### 初始化阶段invokeInitMethods

由AbstractAutowireCapableBeanFactory#invokeInitMethods实现，主要做两件事：

1. 处理 InitializingBean#afterPropertiesSet() 方法
2. 处理自定义的init-method方法

```java
public abstract class AbstractAutowireCapableBeanFactory extends AbstractBeanFactory
      implements AutowireCapableBeanFactory {
	//......
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
				throws Throwable {
	
		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
	//......
}
```

#### 初始化后阶段

初始化后阶段主要通过 AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization() 方法进行实现，用来处理BeanPostProcessor#postProcessAfterInitialization()方法。

这里是否也承担了部分Aware接口回调？这里笔者没有详细进行测试，后续有时间补充。

#### 初始化完成阶段

初始化完成阶段主要通过 SmartInitializingSingleton接口进行实现，这个功能在Spring4.1+版本得到支持。是在 ApplicationContext#refresh()方法中调用了 ApplicationContext#finishBeanFactoryInitialization() 进行实现。

### 销毁阶段

一般在ApplicationContext关闭的时候调用，也就是AbstractApplicationContext#close() 方法。除此以外也可以主动调用销毁方法。

这里在注册的时候Spring通过适配器模式包装了一个类DisposableBeanAdapter，在销毁阶段的时候会获得这个类，进而调用到DisposableBeanAdapter#destroy()方法：

```java
class DisposableBeanAdapter implements DisposableBean, Runnable, Serializable {
	//......
	@Override
	public void destroy() {
		if (!CollectionUtils.isEmpty(this.beanPostProcessors)) {
			for (DestructionAwareBeanPostProcessor processor : this.beanPostProcessors) {
				processor.postProcessBeforeDestruction(this.bean, this.beanName);
			}
		}

		if (this.invokeDisposableBean) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking destroy() on bean with name '" + this.beanName + "'");
			}
			try {
				if (System.getSecurityManager() != null) {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((DisposableBean) this.bean).destroy();
						return null;
					}, this.acc);
				}
				else {
					((DisposableBean) this.bean).destroy();
				}
			}
			catch (Throwable ex) {
				String msg = "Invocation of destroy method failed on bean with name '" + this.beanName + "'";
				if (logger.isDebugEnabled()) {
					logger.warn(msg, ex);
				}
				else {
					logger.warn(msg + ": " + ex);
				}
			}
		}

		if (this.destroyMethod != null) {
			invokeCustomDestroyMethod(this.destroyMethod);
		}
		else if (this.destroyMethodName != null) {
			Method methodToInvoke = determineDestroyMethod(this.destroyMethodName);
			if (methodToInvoke != null) {
				invokeCustomDestroyMethod(ClassUtils.getInterfaceMethodIfPossible(methodToInvoke));
			}
		}
	}
	//......
}
```

##### 销毁前阶段

主要由 DestructionAwareBeanPostProcessor#postProcessBeforeDestruction()提供在销毁前需要执行的方法,从上面的代码中也可以看到。

##### 销毁阶段

这里主要包括三个销毁途径，按照执行顺序有以下：

1. @PreDestroy注解，主要通过DestructionAwareBeanPostProcessor实现
2. 实现DisposableBean接口，主要通过DisposableBean#destroy()实现
3. 自定义销毁方法DisposableBeanAdapter#invokeCustomDestroyMethod()实现

可以看到，Spring的bean的主要生命周期其实就是注册→合并→实例→初始化→销毁，这之中很多的拓展功能是通过各种各样的BeanPostProcessor去实现的，最典型的就是我们的初始化与销毁阶段的几个钩子方法，按照执行顺序，总结如下：

构造：

    @PostConstruct 注解，通过InitDestroyAnnotationBeanPostProcessor实现。
    InitializingBean#afterPropertiesSet方法，在初始化的时候直接调用
    自定义的init-method

销毁：

    @PreDestroy注解，通过DestructionAwareBeanPostProcessor实现
    实现DisposableBean接口，在销毁的时候直接调用
    自定义的destroy-method

总结规律就是，注解优先(通过postProcessor实现)，接口实现其次(直接调用)，自定义最后。

## 案例

### 创建Bean对象

```java
public class PeopleBean implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean {

   private String name;

   private String address;

   private String phone;

   private BeanFactory beanFactory;

   private String beanName;

   public PeopleBean(){
       System.out.println("【构造器】调用Person的构造器实例化");
   }
    public String getName() {
       return name;
   }
    public void setName(String name) {
       System.out.println("【注入属性】注入属性name");
       this.name = name;
   }

    public String getAddress() {
       return address;
   }

    public void setAddress(String address) {
       System.out.println("【注入属性】注入属性address");
       this.address = address;
   }

    public String getPhone() {
       return phone;
   }

    public void setPhone(String phone) {
       System.out.println("【注入属性】注入属性phone");
       this.phone = phone;
   }

    @Override
    public String toString() {
        return "PeopleBean{" +
                "name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", phone='" + phone + '\'' +
                ", beanFactory=" + beanFactory +
                ", beanName='" + beanName + '\'' +
                '}';
    }

    /**
     * BeanFactoryAware
     * @param beanFactory
     * @throws BeansException
     */
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()");
        this.beanFactory = beanFactory;
   }

    /**
     * BeanNameAware
     * @param name
     */
    @Override
    public void setBeanName(String name) {
        System.out.println("【BeanNameAware接口】调用BeanNameAware.setBeanName()");
        this.beanName = name;
    }

    /**
     * DisposableBean
     * @throws Exception
     */
    @Override
    public void destroy() throws Exception {
        System.out.println("【DiposibleBean接口】调用DiposibleBean.destory()");
    }

    /**
     * InitializingBean
     * @throws Exception
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("【InitializingBean接口】调用InitializingBean.afterPropertiesSet()");
    }

    public void beanInit(){
        System.out.println("【init-method】调用<bean>的init-method属性指定的初始化方法");
    }

    public void beanDestory(){
        System.out.println("【destroy-method】调用<bean>的destroy-method属性指定的初始化方法");
    }
}
```

创建Bean对象，实现BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean等接口，xxxAware等接口主要用于感知，如果想获取创建bean对象的工厂对象，或者bean的名字或者获取当前bean的类加载器，都需要实现Aware接口，此接口在创建bean的时候，将加载bean的一些工具注入到容器中，因此在程序上下文中就可以直接获取。InitializingBean是bean对象初始化接口实现，DisposableBean是bean对象销毁接口实现。

### InstantiationAwareBeanPostProcessorAdapter接口实现

```java
public class MyInstantiationAwareBeanPostProcessor extends
        InstantiationAwareBeanPostProcessorAdapter {


    public MyInstantiationAwareBeanPostProcessor(){
        System.out.println("这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！");
    }

    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        System.out.println("InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法");

        return super.postProcessBeforeInstantiation(beanClass, beanName);
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        System.out.println("InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法");

        return super.postProcessAfterInstantiation(bean, beanName);
    }


    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        System.out.println("InstantiationAwareBeanPostProcessor调用postProcessProperties方法");
        return super.postProcessProperties(pvs, bean, beanName);
    }
}
```

InstantiationAwareBeanPostProcessor 接口继承了 BeanPostProcessor 接口。   
BeanPostProcessor 主要是在 Bean 调用初始化方法前后进行拦截。  而 InstantiationAwareBeanPostProcessor 接口在 BeanPostProcessor 基础上添加了对 bean 的创建前后，以及设置属性前进行拦截，接口中主要有下面几个方法：

| 方法                              | 执行顺序                     | 备注                                                         |
| --------------------------------- | ---------------------------- | ------------------------------------------------------------ |
| postProcessBeforeInstantiation()  | 在 Bean 创建前调用           | 可用于创建代理类，如果返回的不是 null(也就是返回的是一个代理类) ，那么后续只会调用 postProcessAfterInitialization() 方法 |
| postProcessAfterInstantiation()   | 在 Bean 创建后调用           | 返回值会影响 postProcessProperties() 是否执行，其中返回 false 的话，是不会执行。 |
| postProcessProperties()           | 在 Bean 设置属性前调用       | 用于修改 bean 的属性，如果返回值不为空，那么会更改指定字段的值 |
| postProcessBeforeInitialization() | 在 Bean 调用初始化方法前调用 | 允许去修改 bean 实例化后，没有调用初始化方法前状态的属性     |
| postProcessAfterInitialization()  | 在 Bean 调用初始化方法后调用 | 允许去修改 bean 调用初始化方法后状态的属性                   |

### 实现BeanPostProcessor接口

```java
public class MyBeanPostProcessor implements BeanPostProcessor {


    public MyBeanPostProcessor(){
        super();
        System.out.println("这是BeanPostProcessor实现类构造器！！");
    }

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！");
        return null;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！");
        return null;
    }
}
```



### 实现BeanFactoryPostProcessor接口

```java
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {

    public MyBeanFactoryPostProcessor(){
        super();
        System.out.println("这是BeanFactoryPostProcessor实现类构造器！！");
    }

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {

        System.out.println("BeanFactoryPostProcessor调用postProcessBeanFactory方法");
        BeanDefinition bd = beanFactory.getBeanDefinition("person");
        bd.getPropertyValues().addPropertyValue("phone", "110");
    }
}
```

BeanFactoryPostProcessor#postProcessBeanFactory可以对BeanDefinition定义进行修改。

### 定义xml文件

```java
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
       xmlns:aop="http://www.springframework.org/schema/aop" xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans-3.2.xsd">

    <bean id="beanPostProcessor" class="bugcode.online.spring.controller.MyBeanPostProcessor">
    </bean>

    <bean id="instantiationAwareBeanPostProcessor" class="bugcode.online.spring.controller.MyInstantiationAwareBeanPostProcessor">
    </bean>

    <bean id="beanFactoryPostProcessor" class="bugcode.online.spring.controller.MyBeanFactoryPostProcessor">
    </bean>

    <bean id="person" class="bugcode.online.spring.domain.PeopleBean" init-method="beanInit"
          destroy-method="beanDestory" scope="singleton" p:name="张三" p:address="广州"
          p:phone="15900000000" />

</beans>
```

### 测试

```java
        System.out.println("现在开始初始化容器");
        ApplicationContext factory = new ClassPathXmlApplicationContext("bean.xml");
        System.out.println("容器初始化成功");
//        得到Preson，并使用
         PeopleBean PeopleBean = factory.getBean("person",PeopleBean.class);
         System.out.println(PeopleBean);
         System.out.println("现在开始关闭容器！");
         ((ClassPathXmlApplicationContext)factory).registerShutdownHook();
```



**结果**

```text
现在开始初始化容器
这是BeanFactoryPostProcessor实现类构造器！！
BeanFactoryPostProcessor调用postProcessBeanFactory方法
这是BeanPostProcessor实现类构造器！！
这是InstantiationAwareBeanPostProcessorAdapter实现类构造器！！
InstantiationAwareBeanPostProcessor调用postProcessBeforeInstantiation方法
【构造器】调用Person的构造器实例化
InstantiationAwareBeanPostProcessor调用postProcessAfterInitialization方法
InstantiationAwareBeanPostProcessor调用postProcessProperties方法
【注入属性】注入属性address
【注入属性】注入属性name
【注入属性】注入属性phone
【BeanNameAware接口】调用BeanNameAware.setBeanName()
【BeanFactoryAware接口】调用BeanFactoryAware.setBeanFactory()
BeanPostProcessor接口方法postProcessBeforeInitialization对属性进行更改！
【InitializingBean接口】调用InitializingBean.afterPropertiesSet()
【init-method】调用<bean>的init-method属性指定的初始化方法
BeanPostProcessor接口方法postProcessAfterInitialization对属性进行更改！
容器初始化成功
PeopleBean{name='张三', address='广州', phone='110', beanFactory=org.springframework.beans.factory.support.DefaultListableBeanFactory@5649fd9b: defining beans [beanPostProcessor,instantiationAwareBeanPostProcessor,beanFactoryPostProcessor,person]; root of factory hierarchy, beanName='person'}
现在开始关闭容器！
【DiposibleBean接口】调用DiposibleBean.destory()
【destroy-method】调用<bean>的destroy-method属性指定的初始化方
```

从打印结果可以看出各个接口在bean创建过程中的调用顺序：

```
1. BeanFactoryPostProcessor：用于对注册的BeanDefinition定义进行修改
MyBeanFactoryPostProcessor#MyBeanFactoryPostProcessor
MyBeanFactoryPostProcessor#postProcessBeanFactory

2. InstantiationAwareBeanPostProcessorAdapter 用户在实例化bean对象前后调用 bean对象实例化
InstantiationAwareBeanPostProcessorAdapter#MyInstantiationAwareBeanPostProcessor
InstantiationAwareBeanPostProcessorAdapter#postProcessBeforeInstantiation
PeopleBean#PeopleBean
InstantiationAwareBeanPostProcessorAdapter#postProcessAfterInstantiation
InstantiationAwareBeanPostProcessorAdapter#postProcessProperties

3 Bean对象属性赋值
PeopleBean#setAddress
PeopleBean#setName
PeopleBean#setPhone
BeanNameAware#setBeanName
BeanFactoryAware#setBeanFactory

4、bean对象初始化
MyBeanPostProcessor#postProcessBeforeInitialization
InitializingBean#afterPropertiesSet
PeopleBean#beanInit
MyBeanPostProcessor#postProcessAfterInitialization

5、bean对象销毁
DisposableBean#destroy
PeopleBean#beanDestory
```



## InstantiationAwareBeanPostProcessor接口

InstantiationAwareBeanPostProcessor接口是BeanPostProcessor的子接口，通过接口字面意思翻译该接口的作用是感知Bean实例话的处理器。实际上该接口的作用也是确实如此。

### 接口定义

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {
	
	@Nullable
	default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
		return null;
	}

	default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
		return true;
	}
	@Nullable
	default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
			throws BeansException {

		return null;
	}

	@Deprecated
	@Nullable
	default PropertyValues postProcessPropertyValues(
			PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

		return pvs;
	}
}

public interface BeanPostProcessor {

	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

InstantiationAwareBeanPostProcessor接口中继承其父类的两个方法，因此此接口一共有5个方法，5个方法的所用分别是：

| postProcessBeforeInitialization | BeanPostProcessor接口中的方法,在Bean的自定义初始化方法之前执行 |
| ------------------------------- | ------------------------------------------------------------ |
| postProcessAfterInitialization  | BeanPostProcessor接口中的方法 在Bean的自定义初始化方法执行完成之后执行 |
| postProcessBeforeInstantiation  | 自身方法，是最先执行的方法，它在目标对象实例化之前调用，该方法的返回值类型是Object，我们可以返回任何类型的值。由于这个时候目标对象还未实例化，所以这个返回值可以用来代替原本该生成的目标对象的实例(比如代理对象)。如果该方法的返回值代替原本该生成的目标对象，后续只有postProcessAfterInitialization方法会调用，其它方法不再调用；否则按照正常的流程走 |
| postProcessAfterInstantiation   | 在目标对象实例化之后调用，这个时候对象已经被实例化，**但是该实例的属性还未被设置**，都是null。因为它的返回值是决定要不要调用postProcessPropertyValues方法的其中一个因素(因为还有一个因素是mbd.getDependencyCheck())；如果该方法返回false,并且不需要check，那么postProcessPropertyValues就会被忽略不执行；如果返回true，postProcessPropertyValues就会被执行 |
| postProcessPropertyValues       | 对属性值进行修改，如果postProcessAfterInstantiation方法返回false，该方法可能不会被调用。可以在该方法内对属性值进行修改 |

> Instantiation:表示实例化,对象还未生成
>
> Initialization:表示初始化,对象已经生成
>
> 对象实例化之后，所有属性值都是类型的默认值，postProcessPropertyValues方法是用来修改默认值的方法。



### 创建Bean定义

```java
@Component
public class BeanObj implements InitializingBean {

    protected Object field;

    public BeanObj(){
        System.out.println("创建 BeanObj 对象");
    }

    public Object getField() {
        return field;
    }

    /**
     * setter注入
     * @param field
     */
    public void setField(Object field) {
        System.out.println("设置 field 属性");
        this.field = field;
    }

    /**
     * 初始化方法
     * @throws Exception
     */
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用 InitializingBean 初始化方法 afterPropertiesSet");
    }

    @Override
    public String toString() {
        return "BeanObj{" +
                "field=" + field +
                '}';
    }

    /**
     * 自定义初始化方法
     */
    public void init(){
        System.out.println("执行 BeanObj 自定义初始化方法init");
    }

    /**
     * 自定义销毁方法
     */
    public void destroy(){
        System.out.println("执行 BeanObj 自定义初始化方法destroy");
    }
}
```

### 实现接口InstantiationAwareBeanPostProcessor

```java
public class CustomInstantiationAwareBeanPostProcessor implements InstantiationAwareBeanPostProcessor {

    /**
     * 在 Bean 创建前调用 可用于创建代理类，如果返回的不是 null（也就是返回的是一个代理类） ，
     *  * 那么后续只会调用 postProcessAfterInitialization() 方法
     * @param beanClass
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        if(isMatchClass(beanClass)){
            System.out.println("调用 postProcessBeforeInstantiation 方法");
        }
        return null;
    }

    /**
     * 在 Bean 创建后调用 返回值会影响 postProcessProperties()
     * 是否执行，其中返回 false 的话，是不会执行。
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if(isMatchClass(bean.getClass())){
            System.out.println("调用 postProcessAfterInstantiation 方法");
        }
        return true;
    }

    /**
     * 在 Bean 设置属性前调用 用于修改 bean 的属性，如果返回值不为空，那么会更改指定字段的值
     * @param pvs
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
        if(isMatchClass(bean.getClass())){
            System.out.println("调用 postProcessProperties 方法");
        }
        return pvs;
    }

    /**
     * 在 Bean 调用初始化方法前调用 允许去修改 bean 实例化后，没有调用初始化方法前状态的属性
     * @param bean
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if(isMatchClass(bean.getClass())){
            System.out.println("调用 postProcessBeforeInitialization 方法");
        }
        return bean;
    }

    /**
     *
     * @param bean 在 Bean 调用初始化方法后调用	允许去修改 bean 调用初始化方法后状态的属性
     * @param beanName
     * @return
     * @throws BeansException
     */
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if(isMatchClass(bean.getClass())){
            System.out.println("调用 postProcessAfterInitialization 方法");
        }
        return bean;
    }

    private boolean isMatchClass(Class<?> beanClass){
        // ClassUtils 类为 org.springframework.util.ClassUtils
        return BeanObj.class.equals(ClassUtils.getUserClass(beanClass));
    }

}
```

**执行结果**

```java
调用 postProcessBeforeInstantiation 方法 创建bean对象前执行
创建 BeanObj 对象 创建bean对象
调用 postProcessAfterInstantiation 方法  bean对象创建后执行
调用 postProcessProperties 方法 设置bean对象的属性值 修改默认值
设置 field 属性 调用setter设置属性值
调用 postProcessBeforeInitialization 方法 调用前置处理器  
调用 InitializingBean 初始化方法 afterPropertiesSet 调用初始化防范
执行 BeanObj 自定义初始化方法init 调用自定义初始化方法
调用 postProcessAfterInitialization 方法 调用后置处理器
执行 BeanObj 自定义初始化方法destroy 调用bean对象销毁方法
```

通过输出结果发现五个方法全部执行，因此接下来我们分析五个方法执行的关联关系；

### postProcessBeforeInstantiation

postProcessBeforeInstantiation() 方法是在 Bean 创建之前调用的，该方法允许我们返回 Bean 的其他子类(我们也可以用 cglib 返回一个代理对象)。  这个方法在 InstantiationAwareBeanPostProcessor 接口的默认实现是返回 null。

**结论**

如果 postProcessBeforeInstantiation() 返回的不是 null ，那么 Spring 只会在 Bean 创建的时候，
只调用 postProcessBeforeInstantiation() 和 postProcessAfterInitialization() 方法。如果返回结果为null，那么后续定义的方法都会执行。

```java
 @Override
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
//        利用cglib动态代理生成对象返回

        if (beanClass == BeanObj.class) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(beanClass);
            enhancer.setCallback(new MethodInterceptor() {
                @Override
                public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                    System.out.println("目标方法执行前："+method);
                    Object invoke = methodProxy.invokeSuper(o, objects);
                    System.out.println("目标方法执行后："+method);
                    return invoke;
                }
            });
           BeanObj o =  (BeanObj)enhancer.create();
           return o;
        }
        return null;
    }
```

在postProcessBeforeInstantiation方法中使用动态代理创建对象并返回。

**执行结果**

```java
调用 postProcessBeforeInstantiation 方法
创建 BeanObj 对象
调用 postProcessAfterInitialization 方法
目标方法执行前：public java.lang.String java.lang.Object.toString()
目标方法执行前：public native int java.lang.Object.hashCode()
目标方法执行后：public native int java.lang.Object.hashCode()
目标方法执行后：public java.lang.String java.lang.Object.toString(
```

从结果可以看到，postProcessBeforeInstantiation方法返回实例对象后跳过了对象的初始化操作，直接执行了postProcessAfterInitialization(该方法在自定义初始化方法执行完成之后执行)，跳过了postProcessAfterInstantiation，postProcessPropertyValues以及自定义的初始化方法(start方法)。

在AbstractBeanFactory中的对InstantiationAwareBeanPostProcessor实现该接口的BeanPostProcessor 设置了标志

```java
@Override
public void addBeanPostProcessor(BeanPostProcessor beanPostProcessor) {
	Assert.notNull(beanPostProcessor, "BeanPostProcessor must not be null");
	this.beanPostProcessors.remove(beanPostProcessor);
	this.beanPostProcessors.add(beanPostProcessor);
	if (beanPostProcessor instanceof InstantiationAwareBeanPostProcessor) {
		this.hasInstantiationAwareBeanPostProcessors = true;
	}
	if (beanPostProcessor instanceof DestructionAwareBeanPostProcessor) {
		this.hasDestructionAwareBeanPostProcessors = true;
	}
}
```

在AbstractAutowireCapableBeanFactory类中有个createBean方法，

```java
protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
       // ... 省略
	try {
		// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
		Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
		if (bean != null) {
				return bean;
		}
       // ... 省略
	Object beanInstance = doCreateBean(beanName, mbdToUse, args);
	if (logger.isDebugEnabled()) {
		logger.debug("Finished creating instance of bean '" + beanName + "'");
	}
	return beanInstance;
}
```

Object bean = resolveBeforeInstantiation(beanName,  mbdToUse);这行代码之后之后根据bean判断如果不为空null就直接返回了，而不执行doCreateBean()方法了，而该方法是创建Bean对象的方法。

```java
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
	Object bean = null;
	// //如果beforeInstantiationResolved还没有设置或者是false（说明还没有需要在实例化前执行的操作）
	if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
		// 判断是否有注册过InstantiationAwareBeanPostProcessor类型的bean
		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
			Class<?> targetType = determineTargetType(beanName, mbd);
			if (targetType != null) {
				bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
				if (bean != null) {
					// 直接执行自定义初始化完成后的方法,跳过了其他几个方法
					bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
				}
			}
		}
		mbd.beforeInstantiationResolved = (bean != null);
	}
	return bean;
}

protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
	for (BeanPostProcessor bp : getBeanPostProcessors()) {
		if (bp instanceof InstantiationAwareBeanPostProcessor) {
			InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
			Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName); 
			//只要有一个result不为null；后面的所有 后置处理器的方法就不执行了，直接返回(所以执行顺序很重要)
			if (result != null) {
				return result;
			}
		}
	}
	return null;
}
```

从 resolveBeforeInstantiation() 方法的代码片段中，它的逻辑基本是这样：

1. 遍历所有的 InstantiationAwareBeanPostProcessor 实现类，调用其 postProcessBeforeInstantiation() 方法。
2. 如果 postProcessBeforeInstantiation() 方法返回值，那么就表示找到了代理类，那么就会结束遍历
3. 如果步骤2没有找到代理类，那么 resolveBeforeInstantiation() 方法就基本执行结束，返回 null 值。
4. 如果步骤2找到了代理类，就会遍历所有 BeanPostProcessor 的实现类，调用其 postProcessAfterInitialization() 方法。
5. 如果 postProcessAfterInitialization() 方法返回的是 null ， 那么就返回上一次遍历得到的结果，或者是代理类本身。

### postProcessAfterInstantiation()

postProcessAfterInstantiation() 方法的返回值会影响 postProcessProperties() 是否会执行：

- false ：不会执行
- true ：执行

原因

上面已经提及了 Spring 创建 Bean 是会调用 createBean() 方法，其中如果 postProcessBeforeInstantiation() 返回的值为 null 的话，就会执行 doCreateBean() 方法。

```java
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    // 忽略上面代码
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        exposedObject = initializeBean(beanName, exposedObject, mbd);
    }
    // 忽略下面代码
}

protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 忽略上面代码
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            // 遍历所有 InstantiationAwareBeanPostProcessor ，并调用其 postProcessAfterInstantiation() 方法，如果返回 false ，就不会执行后续步骤了。
            if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                return;
            }
        }
    }
    // 忽略中间部分代码
    if (hasInstantiationAwareBeanPostProcessors()) {
        if (pvs == null) {
            pvs = mbd.getPropertyValues();
        }
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            // 遍历所有 InstantiationAwareBeanPostProcessor ，并调用其 postProcessProperties() 方法，如果存在返回的值为 null ，就不会执行后续步骤了
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
            pvs = pvsToUse;
        }
    }

    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);
    if (needsDepCheck) {
        PropertyDescriptor[] filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }

    if (pvs != null) {
        // 根据调用所有 InstantiationAwareBeanPostProcessor 的 postProcessProperties() 方法所得到的值，来修改这个 bean 的属性
        applyPropertyValues(beanName, mbd, bw, pvs);
    }
}
```

从上面的代码片段中，doCreateBean() 方法会调用 populateBean() 方法，其中 populateBean() 方法它的主要逻辑是：

1. 遍历所有 InstantiationAwareBeanPostProcessor ，调用 postProcessAfterInstantiation() 方法，如果返回值为 false 了，就不会执行后续步骤了
2. 遍历所有 InstantiationAwareBeanPostProcessor ，调用 postProcessProperties() 方法，如果返回值为 null ，就不会执行后续的其他步骤了
3. 如果 pvs 的值不为空，那么就去调用 applyPropertyValues() 方法来设置 bean 的属性

### postProcessProperties()

postProcessProperties() 方法返回的 PropertyValues 会影响 bean 属性的设置。

结论

如果 PropertyValues 为 null ，PropertyValues 里面没有数据，就不会对 bean 的属性做任何修改。  
否则，就会修改 Bean 的属性值。

```java
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) throws BeansException {
    if (isMatchClass(bean.getClass())) {
        System.out.println("调用 postProcessProperties 方法");
        MutablePropertyValues mpvs = (MutablePropertyValues) pvs;
        // 这里修改 TestBean 的 field 字段值为 "update value"
        mpvs.add("field","update value");
    }
    return pvs;
}
```

### postProcessBeforeInitialization() 和 postProcessAfterInitialization()

postProcessBeforeInitialization() 和 postProcessAfterInitialization() 这2个方法是在 Bean 的属性设置完之后执行的，可以通过这2个方法来完成对 Bean 的属性进行修改。  
当 Spring 在 AbstractAutowireCapableBeanFactory 的 doCreateBean() 方法中调用完了 populateBean() 的方法后，就会调用 initializeBean() 方法。  
故 initializeBean() 方法是 Spring 调用 postProcessBeforeInitialization() 和 postProcessAfterInitialization() 的主要逻辑：

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    invokeAwareMethods(beanName, bean);

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        // 遍历所有 BeanPostProcessor 的子类，并调用 postProcessBeforeInitialization() 的方法
        // 如果 postProcessBeforeInitialization() 返回 null ，则返回上一个 BeanPostProcessor 子类返回的对象，
        // 或者是 bean 对象本身（如果调用第一个 BeanPostProcessor 子类就返回 null 的话）  
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
    }

    try {
        // 调用 bean 指定的 init 方法，或者是实现 InitializingBean 的 afterPropertiesSet() 方法
        invokeInitMethods(beanName, wrappedBean, mbd);
    }
    catch (Throwable ex) {
        throw new BeanCreationException(
                (mbd != null ? mbd.getResourceDescription() : null), beanName, ex.getMessage(), ex);
    }
    if (mbd == null || !mbd.isSynthetic()) {
        // 遍历所有 BeanPostProcessor 的子类，并调用 postProcessAfterInitialization() 的方法
        // 如果 postProcessAfterInitialization() 返回 null ，则返回上一个 BeanPostProcessor 子类返回的对象，
        // 或者是 bean 对象本身（如果调用第一个 BeanPostProcessor 子类就返回 null 的话）  
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```



## 总结

1. InstantiationAwareBeanPostProcessor接口继承BeanPostProcessor接口，它内部提供了3个方法，再加上BeanPostProcessor接口内部的2个方法，所以实现这个接口需要实现5个方法。InstantiationAwareBeanPostProcessor接口的主要作用在于目标对象的实例化过程中需要处理的事情，包括实例化对象的前后过程以及实例的属性设置
2. postProcessBeforeInstantiation方法是最先执行的方法，它在目标对象实例化之前调用，该方法的返回值类型是Object，我们可以返回任何类型的值。由于这个时候目标对象还未实例化，所以这个返回值可以用来代替原本该生成的目标对象的实例(比如代理对象)。如果该方法的返回值代替原本该生成的目标对象，后续只有postProcessAfterInitialization方法会调用，其它方法不再调用；否则按照正常的流程走
3. postProcessAfterInstantiation方法在目标对象实例化之后调用，这个时候对象已经被实例化，但是该实例的属性还未被设置，都是null。因为它的返回值是决定要不要调用postProcessPropertyValues方法的其中一个因素(因为还有一个因素是mbd.getDependencyCheck())；如果该方法返回false,并且不需要check，那么postProcessPropertyValues就会被忽略不执行；如果返回true, postProcessPropertyValues就会被执行
4. postProcessPropertyValues方法对属性值进行修改(这个时候属性值还未被设置，但是我们可以修改原本该设置进去的属性值)。如果postProcessAfterInstantiation方法返回false，该方法可能不会被调用。可以在该方法内对属性值进行修改
5. 父接口BeanPostProcessor的2个方法postProcessBeforeInitialization和postProcessAfterInitialization都是在目标对象被实例化之后，并且属性也被设置之后调用的