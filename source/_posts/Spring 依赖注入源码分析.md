---
title: Spring 依赖注入源码分析
date: 2014-02-13
tags: [Spring]
categories: [Spring 源码分析]
---

# getBean

> 之前写的[Spring IOC 容器](https://vnice.github.io/2014/02/04/Spring%20IOC%20%E5%AE%B9%E5%99%A8/)这篇文章大致讲了一些IoC容器的初始化过程,主要是IoC容器中建立BeanDefinition的数据映射。
但是还没涉及到IoC容器对Bean的依赖关系进行注入，接下来从源码角度分析一下IoC容器是怎样对Bean的依赖关系进行注入的。

首先注意到依赖注入的过程是用户第一次向IoC容器获取Bean时触发的。也就是最基本的BeanFactory接口中的getBean方法，下面从它
`DefaultListableBeanFactory`的基类`AbstractBeanFactory`入手去看下getBean的实现

> 通过查看getBean方法的几个重载，正在干活的是doGetBean方法,也就是依赖注入发生的地方

```java
    protected <T> T doGetBean(
    			final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
    			throws BeansException {
    		final String beanName = transformedBeanName(name);
    		Object bean;

            // 这里先检查缓存中是否已经有单例对象
    		// Eagerly check singleton cache for manually registered singletons.
    		Object sharedInstance = getSingleton(beanName);
    		if (sharedInstance != null && args == null) {
    			if (logger.isDebugEnabled()) {
    			     // 一些健壮性的代码检查单例对象是否还未完成初始化
    				if (isSingletonCurrentlyInCreation(beanName)) {
    					logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
    							"' that is not fully initialized yet - a consequence of a circular reference");
    				}
    				else {
    					logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
    				}
    			}
    			// 如果是单例对象，在这里根据sharedInstance直接返回了，这里包含了一些处理sharedInstance是否是
    			// FactoryBean的逻辑,如果是普通对象直接返回sharedInstance，如果sharedInstance是FactoryBean
    			// 则通过这个FactoryBean来创建一个单例对象返回
    			bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
    		}

    		else {
    		    // 多例状态的bean如果正在创建过程中抛出异常，避免循环引用
    			// Fail if we're already creating this bean instance:
    			// We're assumably within a circular reference.
    			if (isPrototypeCurrentlyInCreation(beanName)) {
    				throw new BeanCurrentlyInCreationException(beanName);
    			}

                //判断BeanDefintion是否存在，顺着BeanFactory链条一直往上递归查找
    			// Check if bean definition exists in this factory.
    			BeanFactory parentBeanFactory = getParentBeanFactory();
    			if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
    				// Not found -> check parent.
    				String nameToLookup = originalBeanName(name);
    				if (args != null) {
    					// Delegation to parent with explicit args.
    					return (T) parentBeanFactory.getBean(nameToLookup, args);
    				}
    				else {
    					// No args -> delegate to standard getBean method.
    					return parentBeanFactory.getBean(nameToLookup, requiredType);
    				}
    			}

    			if (!typeCheckOnly) {
    				markBeanAsCreated(beanName);
    			}

    			try {
    				final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
    				checkMergedBeanDefinition(mbd, beanName, args);
                    // 获取当前Bean的所有依赖Bean，同样是递归调用getBean,直到所有依赖都获取到。
    				// Guarantee initialization of beans that the current bean depends on.
    				String[] dependsOn = mbd.getDependsOn();
    				if (dependsOn != null) {
    					for (String dependsOnBean : dependsOn) {
    						if (isDependent(beanName, dependsOnBean)) {
    							throw new BeanCreationException(mbd.getResourceDescription(), beanName,
    									"Circular depends-on relationship between '" + beanName + "' and '" + dependsOnBean + "'");
    						}
    						registerDependentBean(dependsOnBean, beanName);
    						getBean(dependsOnBean);
    					}
    				}

    				// Create bean instance.
    				if (mbd.isSingleton()) {
    				// 通过调用createBean 方法创建单例Bean,回调函数则是根据不同实现类来处理createBean的逻辑
    					sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
    						@Override
    						public Object getObject() throws BeansException {
    							try {
    								return createBean(beanName, mbd, args);
    							}
    							catch (BeansException ex) {
    								// Explicitly remove instance from singleton cache: It might have been put there
    								// eagerly by the creation process, to allow for circular reference resolution.
    								// Also remove any beans that received a temporary reference to the bean.
    								destroySingleton(beanName);
    								throw ex;
    							}
    						}
    					});
    					bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
    				}
                    // 多例Bean的创建
    				else if (mbd.isPrototype()) {
    					// It's a prototype -> create a new instance.
    					Object prototypeInstance = null;
    					try {
    						beforePrototypeCreation(beanName);
    						prototypeInstance = createBean(beanName, mbd, args);
    					}
    					finally {
    						afterPrototypeCreation(beanName);
    					}
    					bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
    				}

    				else {
    			        // ... 省略一部分代码
    				}
    			}
    			catch (BeansException ex) {
    				cleanupAfterBeanCreationFailure(beanName);
    				throw ex;
    			}
    		}

            // 对创建的Bean进行类型检查
    		// Check if required type matches the type of the actual bean instance.
    		if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
    			try {
    				return getTypeConverter().convertIfNecessary(bean, requiredType);
    			}
    			catch (TypeMismatchException ex) {
    				if (logger.isDebugEnabled()) {
    					logger.debug("Failed to convert bean '" + name + "' to required type [" +
    							ClassUtils.getQualifiedName(requiredType) + "]", ex);
    				}
    				throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
    			}
    		}
    		// 返回包含了依赖关系的Bean
    		return (T) bean;
    	}
```

# createBean

重点来说getBean是依赖注入的起点，之后会调用createBean，下面通过createBean源码来了解这个实现过程
在`AbstractAutowireCapableBeanFactory` 中实现了这个createBean，也就是上面的ObjectFactory中getObject回调方法中的createBean，
createBean 不但要生成了需要的Bean,还对Bean初始化进行了处理，比如实现了BeanDefinition中的init-method和Bean的后置处理，代码如下

```java
	protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
		if (logger.isDebugEnabled()) {
			logger.debug("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

        // 这里判断需要创建的Bean是否可以实例化，这个类是否可以通过类加载来加载
		// Make sure bean class is actually resolved at this point, and
		// clone the bean definition in case of a dynamically resolved Class
		// which cannot be stored in the shared merged bean definition.
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}

		try {
			// Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
			// 如果Bean配置了PostProcessor 那么返回一个代理对象
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

        //这里调用真正干活的方法doCreateBean
		Object beanInstance = doCreateBean(beanName, mbdToUse, args);
		if (logger.isDebugEnabled()) {
			logger.debug("Finished creating instance of bean '" + beanName + "'");
		}
		return beanInstance;
	}

```
具体创建bean的方法

```java
    protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
    		// Instantiate the bean.
    		// 这个Wrapper是用来持有创建出来的Bean对象的
    		BeanWrapper instanceWrapper = null;
    		// 如果是单利则把缓存中的同名的Bean 清除
    		if (mbd.isSingleton()) {
    			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    		}

    		// 真正创建Bean的地方，由createBeanInstance来完成
    		if (instanceWrapper == null) {
    			instanceWrapper = createBeanInstance(beanName, mbd, args);
    		}

    		// ... 省略部分代码

    		// 对Bean进行初始化，依赖注入往往发生在这里，这个exposedObject在初始化完成以后会返回作为依赖注入完成后的Bean
    		// Initialize the bean instance.
    		Object exposedObject = bean;
    		try {
    			populateBean(beanName, mbd, instanceWrapper);
    			if (exposedObject != null) {
    				exposedObject = initializeBean(beanName, exposedObject, mbd);
    			}
    		}
    		catch (Throwable ex) {
    			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
    				throw (BeanCreationException) ex;
    			}
    			else {
    				throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    			}
    		}

    		// ... 省略部分代码

    		return exposedObject;
    	}
```
# createBeanInstance

在上面可以看到与依赖关系特别密切的方法有createBeanInstace和，populateBean，下面分别介绍着两个方法。
在createInstance中生成了Bean所包含的Java对象，这个对象的生成有很多不同方式，可以通过工厂方法，也可以通过容器的autowire特性生成，这些
生成方式都是通过BeanDefinition来指定的。源码如下

```java
    protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
    		// Make sure bean class is actually resolved at this point.
    		    // 确保所创建的Bean可以实例化
    		Class<?> beanClass = resolveBeanClass(mbd, beanName);
    		if (beanClass != null && !Modifier.isPublic(beanClass.getMo
    difiers()) && !mbd.isNonPublicAccessAllowed()) {
    			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
    					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
    		}

          // 如果BeanDefinition有工厂方法配置，则用工厂方法模式创建对象
    		if (mbd.getFactoryMethodName() != null)  {
    			return instantiateUsingFactoryMethod(beanName, mbd, args);
    		}

    		// Shortcut when re-creating the same bean...
    		boolean resolved = false;
    		boolean autowireNecessary = false;
    		if (args == null) {
    			synchronized (mbd.constructorArgumentLock) {
    				if (mbd.resolvedConstructorOrFactoryMethod != null) {
    					resolved = true;
    					autowireNecessary = mbd.constructorArgumentsResolved;
    				}
    			}
    		}
    		if (resolved) {
    			if (autowireNecessary) {
    				return autowireConstructor(beanName, mbd, null, null);
    			}
    			else {
    				return instantiateBean(beanName, mbd);
    			}
    		}

            // 使用构造函数进行实例化
    		// Need to determine the constructor...
    		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
    		if (ctors != null ||
    				mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
    				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
    			return autowireConstructor(beanName, mbd, ctors, args);
    		}

    		// No special handling: simply use no-arg constructor.
    		// 使用默认的实例化策略对Bean进行实例化，也就是使用CGLIB对Bean进行实例化
    		return instantiateBean(beanName, mbd);
    	}

```

最后面使用CGLIB对Bean进行实例化。CGLIB是一个常用的字节码生成器类库，他提供了一些API来提供生成和转换Java的字节码功能。
在Spring AOP中也使用CGLIB对Java的字节码进行增强。要了解怎样使用CGLIB来生成Bean对象，需要看下`SimpleInstantitionStrategy`类的源码
这个策略是Spring用来生成Bean对象的默认类，它提供了两种实例化Java对象的方法，一种是BeanUtils,通过反射功能来实例化对象，一种是通过CGLIB来生成

# populateBean
上面个介绍了Bean对象的生成，在Bean对象生成完成以后，怎样把Bean的依赖对象的关系设置好，完成整个依赖注入过程。就是上面提到了populateBean方法，
这里代码优点多不贴了。总之是对各种数据类型的依赖进行注入。

# 参考
[《Spring 技术内幕-深入理解SPring架构与设计原理第二版》](https://item.jd.com/10922251.html)