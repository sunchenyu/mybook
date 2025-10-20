# spring bean的生命周期

## 一、基础

Bean：Bean是spring容器管理的对象。 BD（BeanDefinition）：BD是对Bean的抽象。

**个人理解：** BD 之于 Bean 就像 Class\<?> 之于 Object。 Class\<?> 是对Object的抽象。 BD是对Bean的抽象。

**bean 最基本的流程**

&#x20;1、实例化：即new一个对象的操作&#x20;

2、属性赋值：给对象成员变量赋值&#x20;

3、初始化：在bean的生命周期当中，初始化方法是这两个：&#x20;

（1）配置文件中配置init-method方法&#x20;

（2）实现InitializingBean接口的类中的afterPropertiesSet方法&#x20;

4、销毁：在bean的生命周期当中，销毁方法是这两个：&#x20;

（1）配置文件中配置destroy-method方法&#x20;

（2）实现DisposableBean接口的类中的destroy方法

**spring 提供了对上述流程的扩展**&#x20;

（1）InstantiationAwareBeanPostProcessor接口 =》 实例化前后的扩展&#x20;

（2）BeanPostProcessor接口 =》 初始化前后的扩展

## 二、流程图

![流程图](<../.gitbook/assets/image (23).png>)

## 三、源码分析

**（1）bean的创建会走到AbstractAutowireCapableBeanFactory的createBean方法。**

```
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) throws BeanCreationException {
    if (this.logger.isTraceEnabled()) {
        this.logger.trace("Creating instance of bean '" + beanName + "'");
    }

    RootBeanDefinition mbdToUse = mbd;
    // 拿到bean描述的对象的Class对象
    Class<?> resolvedClass = this.resolveBeanClass(mbd, beanName, new Class[0]);
    if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
        mbdToUse = new RootBeanDefinition(mbd);
        mbdToUse.setBeanClass(resolvedClass);
    }

    try {
        // 验证以及准备需要覆盖的方法
        mbdToUse.prepareMethodOverrides();
    } catch (BeanDefinitionValidationException var9) {
        throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(), beanName, "Validation of method overrides failed", var9);
    }

    Object beanInstance;
    try {
        // bean实例化前可以调用的后置处理器
        // postProcessBeforeInstantiation方法
        beanInstance = this.resolveBeforeInstantiation(beanName, mbdToUse);
        if (beanInstance != null) {
            return beanInstance;
        }
    } catch (Throwable var10) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "BeanPostProcessor before instantiation of bean failed", var10);
    }

    try {
        //bean开始的生命周期
        beanInstance = this.doCreateBean(beanName, mbdToUse, args);
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Finished creating instance of bean '" + beanName + "'");
        }

        return beanInstance;
    } catch (ImplicitlyAppearedSingletonException | BeanCreationException var7) {
        throw var7;
    } catch (Throwable var8) {
        throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", var8);
    }
}
```

**（2）resolveBeforeInstantiation方法**&#x20;

实例化前的扩展：InstantiationAwareBeanPostProcessor接口的postProcessBeforeInstantiation方法

```
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
    Object bean = null;
    if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
        if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
            Class<?> targetType = this.determineTargetType(beanName, mbd);
            if (targetType != null) {
                bean = this.applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
                if (bean != null) {
                    bean = this.applyBeanPostProcessorsAfterInitialization(bean, beanName);
                }
            }
        }

        mbd.beforeInstantiationResolved = bean != null;
    }

    return bean;
}
```

**（3）之后开始走bean的生命周期，doCreateBean方法**

&#x20;1、实例化： instanceWrapper = this.createBeanInstance(beanName, mbd, args);&#x20;

2、属性赋值（含扩展方法）： this.populateBean(beanName, mbd, instanceWrapper);&#x20;

3、初始化（含扩展方法）： exposedObject = this.initializeBean(beanName, exposedObject, mbd);

```
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
 throws BeanCreationException {

    BeanWrapper instanceWrapper = null;
    if (instanceWrapper == null) {
        //实例化阶段 拿到BeanWrapper
        instanceWrapper = this.createBeanInstance(beanName, mbd, args);
    }
    //根据BeanWrapper拿到实例化的对象
    Object bean = instanceWrapper.getWrappedInstance();

    Object exposedObject = bean;
    try {
        //属性赋值阶段（这个方法的最开始，进行了实例化后调用方法的扩展点）
        this.populateBean(beanName, mbd, instanceWrapper);
        //初始化阶段（这个方法的最开始，进行了初始化前的调用方法的扩展点）
        exposedObject = this.initializeBean(beanName, exposedObject, mbd);
    } catch (Throwable var18) {
    }
    return exposedObject;
}
```

**（4）createBeanInstance方法** 实例化过程

```
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
   // Make sure bean class is actually resolved at this point.
   Class<?> beanClass = resolveBeanClass(mbd, beanName);

   if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
   }

   Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
   if (instanceSupplier != null) {
      return obtainFromSupplier(instanceSupplier, beanName);
   }

   if (mbd.getFactoryMethodName() != null) {
      return instantiateUsingFactoryMethod(beanName, mbd, args);
   }

   // 一个类可能有多个构造器，所以Spring得根据参数个数、类型确定需要调用的构造器
   // 在使用构造器创建实例后，Spring会将解析过后确定下来的构造器或工厂方法保存在缓存中，避免再次创建相同bean时再次解析
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

   // Candidate constructors for autowiring?
   Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
   if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
         mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
      return autowireConstructor(beanName, mbd, ctors, args);
   }

   // Preferred constructors for default construction?
   ctors = mbd.getPreferredConstructors();
   if (ctors != null) {
      return autowireConstructor(beanName, mbd, ctors, null);
   }

   // No special handling: simply use no-arg constructor.
   return instantiateBean(beanName, mbd);
}
```

**（5）populateBean方法**&#x20;

两个过程：&#x20;

1、实例化后的扩展：InstantiationAwareBeanPostProcessor接口的postProcessAfterInstantiation方法 2、属性赋值

```
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {

    // 调用postProcessAfterInstantiation
    // 实例化后的扩展点
    if (!mbd.isSynthetic() && this.hasInstantiationAwareBeanPostProcessors()) {
        Iterator var4 = this.getBeanPostProcessors().iterator();
        while(var4.hasNext()) {
            BeanPostProcessor bp = (BeanPostProcessor)var4.next();
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                    return;
                }
            }
        }
    }

    // 以下是属性赋值的过程
    PropertyValues pvs = mbd.hasPropertyValues() ? mbd.getPropertyValues() : null;
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == 1 || resolvedAutowireMode == 2) {
        MutablePropertyValues newPvs = new MutablePropertyValues((PropertyValues)pvs);
        if (resolvedAutowireMode == 1) {
            this.autowireByName(beanName, mbd, bw, newPvs);
        }

        if (resolvedAutowireMode == 2) {
            this.autowireByType(beanName, mbd, bw, newPvs);
        }

        pvs = newPvs;
    }

    boolean hasInstAwareBpps = this.hasInstantiationAwareBeanPostProcessors();
    boolean needsDepCheck = mbd.getDependencyCheck() != 0;
    PropertyDescriptor[] filteredPds = null;
    if (hasInstAwareBpps) {
        if (pvs == null) {
            pvs = mbd.getPropertyValues();
        }

        Iterator var9 = this.getBeanPostProcessors().iterator();

        while(var9.hasNext()) {
            BeanPostProcessor bp = (BeanPostProcessor)var9.next();
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
                InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor)bp;
                PropertyValues pvsToUse = ibp.postProcessProperties((PropertyValues)pvs, bw.getWrappedInstance(), beanName);
                if (pvsToUse == null) {
                    if (filteredPds == null) {
                        filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
                    }

                    pvsToUse = ibp.postProcessPropertyValues((PropertyValues)pvs, filteredPds, bw.getWrappedInstance(), beanName);
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
            filteredPds = this.filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        }

        this.checkDependencies(beanName, mbd, filteredPds, (PropertyValues)pvs);
    }

    if (pvs != null) {
        this.applyPropertyValues(beanName, mbd, bw, (PropertyValues)pvs);
    }

}
```

**（6）initializeBean方法**&#x20;

三个过程：&#x20;

1、初始化前的扩展：BeanPostProcessor接口的postProcessBeforeInitialization方法&#x20;

2、初始化过程：invokeInitMethods&#x20;

3、初始化后的扩展：BeanPostProcessor接口的postProcessAfterInitialization方法

```
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    //让这个bean对象，可以拿到BeanName、 BeanClassLoader、BeanFactory
    this.invokeAwareMethods(beanName, bean);

    Object wrappedBean = bean;
    if (mbd == null || !mbd.isSynthetic()) {
        //初始化前扩展点
        wrappedBean = this.applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    }

    try {
        //调用初始化方法
        this.invokeInitMethods(beanName, wrappedBean, mbd);
    } catch (Throwable var6) {
        throw new BeanCreationException(mbd != null ? mbd.getResourceDescription() : null, beanName, "Invocation of init method failed", var6);
    }

    if (mbd == null || !mbd.isSynthetic()) {
        //初始化后扩展点
        wrappedBean = this.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }

    return wrappedBean;
}
```

**（7）invokeInitMethods方法、bean的初始化方法**

&#x20;两个过程：&#x20;

1、执行InitializingBean 的 afterPropertiesSet 方法&#x20;

2、执行initMethod方法

```
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd) throws Throwable {
    boolean isInitializingBean = bean instanceof InitializingBean;
    if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
        if (this.logger.isTraceEnabled()) {
            this.logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
        }

        if (System.getSecurityManager() != null) {
            try {
                AccessController.doPrivileged(() -> {
                    ((InitializingBean)bean).afterPropertiesSet();
                    return null;
                }, this.getAccessControlContext());
            } catch (PrivilegedActionException var6) {
                throw var6.getException();
            }
        } else {
            // 执行InitializingBean 的 afterPropertiesSet 方法
            ((InitializingBean)bean).afterPropertiesSet();
        }
    }

    // 如果配置了initMethod 就执行initMethod方法
    if (mbd != null && bean.getClass() != NullBean.class) {
        String initMethodName = mbd.getInitMethodName();
        if (StringUtils.hasLength(initMethodName) && (!isInitializingBean || !"afterPropertiesSet".equals(initMethodName)) && !mbd.isExternallyManagedInitMethod(initMethodName)) {
            this.invokeCustomInitMethod(beanName, bean, mbd);
        }
    }
}
```

## 四、测试

在spring boot 当中添加一些类，进行测试

### 1、基础类

```
package com.suncy.test.extension.bfpp;

import org.springframework.beans.factory.BeanNameAware;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.annotation.Value;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class MyCustom implements InitializingBean, DisposableBean, BeanNameAware {
    MyCustom(){
        System.out.println("实例化");
    }

    @PostConstruct
    public void init(){
        System.out.println("PostConstruct注解方法");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("初始化：InitializingBean的afterPropertiesSet方法");
    }

    @PreDestroy
    public void preDestroy(){
        System.out.println("PreDestroy注解方法");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("销毁：DisposableBean接口的destroy方法");
    }

    @Override
    public void setBeanName(String s) {
        System.out.println("Aware：BeanNameAware");
    }

    public void testInit() {
        System.out.println("初始化：bean.xml配置的init-method");
    }

    public void testDe()
    {
        System.out.println("销毁：bean.xml配置的destroy-method");
    }

}
```

### 2、扩展类

BeanPostProcessor扩展

```
package com.suncy.test.extension.bpp;

import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.BeanPostProcessor;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;

@Component
public class Instance implements BeanPostProcessor {
    @Override
    @Nullable
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("MyCustom")) {
            System.out.println("扩展：postProcessBeforeInitialization");
        }
        return bean;
    }

    @Override
    @Nullable
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("MyCustom")) {
            System.out.println("扩展：postProcessAfterInitialization");
        }
        return bean;
    }
}
```

InstantiationAwareBeanPostProcessor扩展

```
package com.suncy.test.extension.bpp;

import com.suncy.test.extension.bfpp.MyCustom;
import org.springframework.beans.BeansException;
import org.springframework.beans.factory.config.InstantiationAwareBeanPostProcessor;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;

@Component
public class InstanceBefore implements InstantiationAwareBeanPostProcessor {
    @Override
    @Nullable
    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
        //每个类都会掉一遍这个 所以需要进行判断，找到我们需要的类
        if (beanClass.equals(MyCustom.class)) {
            System.out.println("扩展：postProcessBeforeInstantiation");
        }

        return null;
    }

    @Override
    public boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
        if (beanName.equals("MyCustom")) {
            System.out.println("扩展：postProcessAfterInstantiation");
        }
        return true;
    }
}
```

### 3、添加xml配置和修改主类

Application入口

```
package com.suncy.test;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.context.annotation.ImportResource;
import org.springframework.scheduling.annotation.EnableAsync;

@EnableAsync
@ImportResource(locations={"classpath:bean.xml"})
@SpringBootApplication(scanBasePackages = "com.suncy")
public class Application {

    public static void main(String[] args) {
        ConfigurableApplicationContext configurableApplicationContext = SpringApplication.run(Application.class, args);
        configurableApplicationContext.close();
    }

}
```

bean.xml

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean  id="MyCustom" class="com.suncy.test.extension.bfpp.MyCustom" init-method="testInit" destroy-method="testDe"></bean>

</beans>
```

### 4、测试结果

![测试结果](<../.gitbook/assets/image (32).png>)

## 五、总结

1、每个bean都要走一遍InstantiationAwareBeanPostProcessor接口和BeanPostProcessor接口，所以如果你需要给特定的bean设置扩展，需要通过方法参数进行类型判断或者beanName判断。
