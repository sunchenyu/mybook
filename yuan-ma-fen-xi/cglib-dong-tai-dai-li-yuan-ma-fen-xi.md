# cglib动态代理源码分析

## 一、用法

### 1、普通类

```
package com.suncy.article.article11.cglib;

public class Hello1{
    public void sayHi1() {
        System.out.println("hi1");
    }

    public void sayHi2() {
        System.out.println("hi2");
    }

    public void sayNo(){
        System.out.println("no");
    }
}
```

### 2、执行代理对象方法的前后完成自己的操作

```
package com.suncy.article.article11.cglib;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class CglibProxy implements MethodInterceptor {
    //需要代理的目标对象
    private Object target;

    //重写拦截方法
    @Override
    public Object intercept(Object obj, Method method, Object[] arr, MethodProxy proxy) throws Throwable {
        System.out.println("cglib before");

        //方法执行，参数：target 目标对象 arr参数数组
        Object invoke = method.invoke(target, arr);

        System.out.println("cglib after");
        return invoke;
    }

    //定义获取代理对象方法
    public Object getCglibProxy(Object objectTarget) {
        //为目标对象target赋值
        this.target = objectTarget;
        //同时可以在对应目录下找到.class文件
        System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\work\\tool\\");
        Enhancer enhancer = new Enhancer();

        //设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
        enhancer.setSuperclass(objectTarget.getClass());

        // 设置回调
        enhancer.setCallback(this);

        //创建并返回代理对象
        Object result = enhancer.create();
        return result;
    }
}
```

### 3、使用

```
package com.suncy.article.article11.cglib;

public class CglibTest {
    public static void main(String[] args) {
        //实例化CglibProxy对象
        CglibProxy cglib = new CglibProxy();
        //获取代理对象
        Hello1 hello1 =  (Hello1) cglib.getCglibProxy(new Hello1());
        //调用代理对象方法
        hello1.sayHi1();
        System.out.println("==========");
        hello1.sayHi2();
        System.out.println("==========");
        hello1.sayNo();
    }
}
```

### 4、测试结果

![测试结果](<../.gitbook/assets/image (13).png>)

### 5、结果说明

cglib是将被代理类所有的方法都进行了拦截，加上了我们自定义的操作。

## 二、源码分析

### 关键点

setSuperclass和setCallback都属于赋值操作，所以关键点在于enhancer.create()方法，这个方法的作用是：生成代理对象。

```
Enhancer enhancer = new Enhancer();

//设置父类,因为Cglib是针对指定的类生成一个子类，所以需要指定父类
enhancer.setSuperclass(objectTarget.getClass());

// 设置回调
enhancer.setCallback(this);

//创建并返回代理对象
Object result = enhancer.create();

//返回代理对象
return result;
```

**1、enhancer.create()**

```
public Object create() {
    classOnly = false;
    argumentTypes = null;
    return createHelper();
}
```

**2、进入createHelper()方法**&#x20;

先是生成了一个key，然后调用了父类的create方法。

```
private Object createHelper() {
    preValidate();
    Object key = KEY_FACTORY.newInstance((superclass != null) ? superclass.getName() : null,
            ReflectUtils.getNames(interfaces),
            filter == ALL_ZERO ? null : new WeakCacheKey<CallbackFilter>(filter),
            callbackTypes,
            useFactory,
            interceptDuringConstruction,
            serialVersionUID);
    this.currentKey = key;
    Object result = super.create(key);
    return result;
}
```

**3、super.create(key)**

```
protected Object create(Object key) {
    try {
        ClassLoader loader = getClassLoader();
        Map<ClassLoader, ClassLoaderData> cache = CACHE;
        ClassLoaderData data = cache.get(loader);
        if (data == null) {
            synchronized (AbstractClassGenerator.class) {
                cache = CACHE;
                data = cache.get(loader);
                if (data == null) {
                    Map<ClassLoader, ClassLoaderData> newCache = new WeakHashMap<ClassLoader, ClassLoaderData>(cache);
                    data = new ClassLoaderData(loader);
                    newCache.put(loader, data);
                    CACHE = newCache;
                }
            }
        }
        this.key = key;
        Object obj = data.get(this, getUseCache());
        if (obj instanceof Class) {
            return firstInstance((Class) obj);
        }
        return nextInstance(obj);
    } catch (RuntimeException e) {
        throw e;
    } catch (Error e) {
        throw e;
    } catch (Exception e) {
        throw new CodeGenerationException(e);
    }
}
```

到这个方法，我们需要打个断点，调试一下，看一下返回值走到的是哪个。&#x20;

![断点调试](<../.gitbook/assets/image (37).png>)

也就是说，通过firstInstance((Class) obj) 方法返回了一个代理对象，而这个方法的入参，我们发现是一个Class，那现在就需要去找到这个Class的涞源。&#x20;

**4、找到Class obj的涞源** 简化代码如下：

```
data = new ClassLoaderData(loader);
Object obj = data.get(this, getUseCache());
```

也就是说这个obj是通过调用data的get方法来的，那么我们去找一下这个get方法。

**5、data.get()方法**

```
  public Object get(AbstractClassGenerator gen, boolean useCache) {
            if (!useCache) {
              return gen.generate(ClassLoaderData.this);
            } else {
              Object cachedValue = generatedClasses.get(gen);
              return gen.unwrapCachedValue(cachedValue);
            }
        }
```

我们需要打个断点，找到进入的是哪个代码块&#x20;

![断点调试](<../.gitbook/assets/image (21).png>)

**6、generatedClasses.get()方法**&#x20;

别忘了这个方法返回的对象是一个Class

```
    public V get(K key) {
        final KK cacheKey = keyMapper.apply(key);
        Object v = map.get(cacheKey);
        if (v != null && !(v instanceof FutureTask)) {
            return (V) v;
        }

        return createEntry(key, cacheKey, v);
    }
```

**7、createEntry(key, cacheKey, v)方法**

```
protected V createEntry(final K key, KK cacheKey, Object v) {
        FutureTask<V> task;
        boolean creator = false;
        if (v != null) {
            // Another thread is already loading an instance
            task = (FutureTask<V>) v;
        } else {
            task = new FutureTask<V>(new Callable<V>() {
                public V call() throws Exception {
                    return loader.apply(key);
                }
            });
            Object prevTask = map.putIfAbsent(cacheKey, task);
            if (prevTask == null) {
                // creator does the load
                creator = true;
                task.run();
            } else if (prevTask instanceof FutureTask) {
                task = (FutureTask<V>) prevTask;
            } else {
                return (V) prevTask;
            }
        }

        V result;
        try {
            result = task.get();
        } catch (InterruptedException e) {
            throw new IllegalStateException("Interrupted while loading cache item", e);
        } catch (ExecutionException e) {
            Throwable cause = e.getCause();
            if (cause instanceof RuntimeException) {
                throw ((RuntimeException) cause);
            }
            throw new IllegalStateException("Unable to load cache item", cause);
        }
        if (creator) {
            map.put(cacheKey, result);
        }
        return result;
    }
```

这个函数也比较长，简化一下

```
task = new FutureTask<V>(new Callable<V>() {
                public V call() throws Exception {
                    return loader.apply(key);
                }
            });
result = task.get();
return result;
```

继续打个断点看一下&#x20;

![](<../.gitbook/assets/image (24).png>)

继续进入&#x20;

![](<../.gitbook/assets/image (45).png>)

到这个地方基本上说明我们通过generatedClasses.get()方法，拿到的东西就是通过这个 gen.generate(ClassLoaderData.this)返回的。

**8、generate方法**

```
 protected Class generate(ClassLoaderData data) {
        Class gen;
        Object save = CURRENT.get();
        CURRENT.set(this);
        try {
            ClassLoader classLoader = data.getClassLoader();
            if (classLoader == null) {
                throw new IllegalStateException("ClassLoader is null while trying to define class " +
                        getClassName() + ". It seems that the loader has been expired from a weak reference somehow. " +
                        "Please file an issue at cglib's issue tracker.");
            }
            synchronized (classLoader) {
              String name = generateClassName(data.getUniqueNamePredicate());              
              data.reserveName(name);
              this.setClassName(name);
            }
            if (attemptLoad) {
                try {
                    gen = classLoader.loadClass(getClassName());
                    return gen;
                } catch (ClassNotFoundException e) {
                    // ignore
                }
            }
            byte[] b = strategy.generate(this);
            String className = ClassNameReader.getClassName(new ClassReader(b));
            ProtectionDomain protectionDomain = getProtectionDomain();
            synchronized (classLoader) { // just in case
                if (protectionDomain == null) {
                    gen = ReflectUtils.defineClass(className, b, classLoader);
                } else {
                    gen = ReflectUtils.defineClass(className, b, classLoader, protectionDomain);
                }
            }
            return gen;
        } catch (RuntimeException e) {
            throw e;
        } catch (Error e) {
            throw e;
        } catch (Exception e) {
            throw new CodeGenerationException(e);
        } finally {
            CURRENT.set(save);
        }
    }
```

我们发现有两个关键的地方，简化一下

```
byte[] b = strategy.generate(this);
 if (protectionDomain == null) {
     gen = ReflectUtils.defineClass(className, b, classLoader);
} else {
     gen = ReflectUtils.defineClass(className, b, classLoader, protectionDomain);
 }
return gen;
```

**9、strategy.generate(this)方法** 我们发现这个方法返回的是一堆byte\[]，可以大胆猜测，这堆byte就是生成的代理类的字节码的详细数据。

```
    public byte[] generate(ClassGenerator cg) throws Exception {
        DebuggingClassWriter cw = getClassVisitor();
        transform(cg).generateClass(cw);
        return transform(cw.toByteArray());
    }
```

打个断点 ![](https://upload-images.jianshu.io/upload_images/26009612-104259ba5f2f20d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](<../.gitbook/assets/image (7) (1).png>)

我们发现generateClass走到的是KeyFactory里的generateClass方法

**10、KeyFactory的generateClass方法**

```
  public void generateClass(ClassVisitor v) {
            ClassEmitter ce = new ClassEmitter(v);

            Method newInstance = ReflectUtils.findNewInstance(keyInterface);
            if (!newInstance.getReturnType().equals(Object.class)) {
                throw new IllegalArgumentException("newInstance method must return Object");
            }

            Type[] parameterTypes = TypeUtils.getTypes(newInstance.getParameterTypes());
            ce.begin_class(Constants.V1_8,
                           Constants.ACC_PUBLIC,
                           getClassName(),
                           KEY_FACTORY,
                           new Type[]{ Type.getType(keyInterface) },
                           Constants.SOURCE_FILE);
            EmitUtils.null_constructor(ce);
            EmitUtils.factory_method(ce, ReflectUtils.getSignature(newInstance));

            int seed = 0;
            CodeEmitter e = ce.begin_method(Constants.ACC_PUBLIC,
                                            TypeUtils.parseConstructor(parameterTypes),
                                            null);
            e.load_this();
            e.super_invoke_constructor();
            e.load_this();
            List<FieldTypeCustomizer> fieldTypeCustomizers = getCustomizers(FieldTypeCustomizer.class);
            for (int i = 0; i < parameterTypes.length; i++) {
                Type parameterType = parameterTypes[i];
                Type fieldType = parameterType;
                for (FieldTypeCustomizer customizer : fieldTypeCustomizers) {
                    fieldType = customizer.getOutType(i, fieldType);
                }
                seed += fieldType.hashCode();
                ce.declare_field(Constants.ACC_PRIVATE | Constants.ACC_FINAL,
                                 getFieldName(i),
                                 fieldType,
                                 null);
                e.dup();
                e.load_arg(i);
                for (FieldTypeCustomizer customizer : fieldTypeCustomizers) {
                    customizer.customize(e, i, parameterType);
                }
                e.putfield(getFieldName(i));
            }
            e.return_value();
            e.end_method();

            // hash code
            e = ce.begin_method(Constants.ACC_PUBLIC, HASH_CODE, null);
            int hc = (constant != 0) ? constant : PRIMES[(int)(Math.abs(seed) % PRIMES.length)];
            int hm = (multiplier != 0) ? multiplier : PRIMES[(int)(Math.abs(seed * 13) % PRIMES.length)];
            e.push(hc);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.hash_code(e, parameterTypes[i], hm, customizers);
            }
            e.return_value();
            e.end_method();

            // equals
            e = ce.begin_method(Constants.ACC_PUBLIC, EQUALS, null);
            Label fail = e.make_label();
            e.load_arg(0);
            e.instance_of_this();
            e.if_jump(e.EQ, fail);
            for (int i = 0; i < parameterTypes.length; i++) {
                e.load_this();
                e.getfield(getFieldName(i));
                e.load_arg(0);
                e.checkcast_this();
                e.getfield(getFieldName(i));
                EmitUtils.not_equals(e, parameterTypes[i], fail, customizers);
            }
            e.push(1);
            e.return_value();
            e.mark(fail);
            e.push(0);
            e.return_value();
            e.end_method();

            // toString
            e = ce.begin_method(Constants.ACC_PUBLIC, TO_STRING, null);
            e.new_instance(Constants.TYPE_STRING_BUFFER);
            e.dup();
            e.invoke_constructor(Constants.TYPE_STRING_BUFFER);
            for (int i = 0; i < parameterTypes.length; i++) {
                if (i > 0) {
                    e.push(", ");
                    e.invoke_virtual(Constants.TYPE_STRING_BUFFER, APPEND_STRING);
                }
                e.load_this();
                e.getfield(getFieldName(i));
                EmitUtils.append_string(e, parameterTypes[i], EmitUtils.DEFAULT_DELIMITERS, customizers);
            }
            e.invoke_virtual(Constants.TYPE_STRING_BUFFER, TO_STRING);
            e.return_value();
            e.end_method();

            ce.end_class();
        }
```

**11、transform(cw.toByteArray())**&#x20;

查看返回值：return transform(cw.toByteArray()); 这个toByteArray方法就是获得了一堆byte

```
public byte[] toByteArray() {

      return (byte[]) java.security.AccessController.doPrivileged(
        new java.security.PrivilegedAction() {
            public Object run() {
                byte[] b = ((ClassWriter) DebuggingClassWriter.super.cv).toByteArray();
                if (debugLocation != null) {
                    String dirs = className.replace('.', File.separatorChar);
                    try {
                        new File(debugLocation + File.separatorChar + dirs).getParentFile().mkdirs();

                        File file = new File(new File(debugLocation), dirs + ".class");
                        OutputStream out = new BufferedOutputStream(new FileOutputStream(file));
                        try {
                            out.write(b);
                        } finally {
                            out.close();
                        }

                        if (traceCtor != null) {
                            file = new File(new File(debugLocation), dirs + ".asm");
                            out = new BufferedOutputStream(new FileOutputStream(file));
                            try {
                                ClassReader cr = new ClassReader(b);
                                PrintWriter pw = new PrintWriter(new OutputStreamWriter(out));
                                ClassVisitor tcv = (ClassVisitor)traceCtor.newInstance(new Object[]{null, pw});
                                cr.accept(tcv, 0);
                                pw.flush();
                            } finally {
                                out.close();
                            }
                        }
                    } catch (Exception e) {
                        throw new CodeGenerationException(e);
                    }
                }
                return b;
             }  
            });
        }
```

其实到这个地方，我们已经能明白cglib代理与jdk代理的相似的地方，那就是都需要先手动构造出一个类的完整的字节码数据，然后将这堆字节码数据加载成为.class文件，之后通过反射生成代理对象。

## 三、查看代理对象的结构

### 1、生成代理Class文件

以下设置将生成的.class文件放入到了tool目录下

```
System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "D:\\work\\tool\\");
```

如图所示&#x20;

![生成的代理class文件](<../.gitbook/assets/image (20).png>)

### 2、查看反编译后的Class代码

下载并安装JD-GUI工具、打开，并将文件拖入即可

```
package com.suncy.article.article11.cglib;

import java.lang.reflect.Method;
import net.sf.cglib.core.ReflectUtils;
import net.sf.cglib.core.Signature;
import net.sf.cglib.proxy.Callback;
import net.sf.cglib.proxy.Factory;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class Hello1$$EnhancerByCGLIB$$3c92030b extends Hello1 implements Factory {
  private boolean CGLIB$BOUND;

  public static Object CGLIB$FACTORY_DATA;

  private static final ThreadLocal CGLIB$THREAD_CALLBACKS;

  private static final Callback[] CGLIB$STATIC_CALLBACKS;

  private MethodInterceptor CGLIB$CALLBACK_0;

  private static Object CGLIB$CALLBACK_FILTER;

  private static final Method CGLIB$sayHi1$0$Method;

  private static final MethodProxy CGLIB$sayHi1$0$Proxy;

  private static final Object[] CGLIB$emptyArgs;

  private static final Method CGLIB$sayHi2$1$Method;

  private static final MethodProxy CGLIB$sayHi2$1$Proxy;

  private static final Method CGLIB$sayNo$2$Method;

  private static final MethodProxy CGLIB$sayNo$2$Proxy;

  private static final Method CGLIB$equals$3$Method;

  private static final MethodProxy CGLIB$equals$3$Proxy;

  private static final Method CGLIB$toString$4$Method;

  private static final MethodProxy CGLIB$toString$4$Proxy;

  private static final Method CGLIB$hashCode$5$Method;

  private static final MethodProxy CGLIB$hashCode$5$Proxy;

  private static final Method CGLIB$clone$6$Method;

  private static final MethodProxy CGLIB$clone$6$Proxy;

  static void CGLIB$STATICHOOK1() {
    CGLIB$THREAD_CALLBACKS = new ThreadLocal();
    CGLIB$emptyArgs = new Object[0];
    Class<?> clazz1 = Class.forName("com.suncy.article.article11.cglib.Hello1$$EnhancerByCGLIB$$3c92030b");
    Class<?> clazz2;
    CGLIB$sayHi1$0$Method = ReflectUtils.findMethods(new String[] { "sayHi1", "()V", "sayHi2", "()V", "sayNo", "()V" }, (clazz2 = Class.forName("com.suncy.article.article11.cglib.Hello1")).getDeclaredMethods())[0];
    CGLIB$sayHi1$0$Proxy = MethodProxy.create(clazz2, clazz1, "()V", "sayHi1", "CGLIB$sayHi1$0");
    CGLIB$sayHi2$1$Method = ReflectUtils.findMethods(new String[] { "sayHi1", "()V", "sayHi2", "()V", "sayNo", "()V" }, (clazz2 = Class.forName("com.suncy.article.article11.cglib.Hello1")).getDeclaredMethods())[1];
    CGLIB$sayHi2$1$Proxy = MethodProxy.create(clazz2, clazz1, "()V", "sayHi2", "CGLIB$sayHi2$1");
    CGLIB$sayNo$2$Method = ReflectUtils.findMethods(new String[] { "sayHi1", "()V", "sayHi2", "()V", "sayNo", "()V" }, (clazz2 = Class.forName("com.suncy.article.article11.cglib.Hello1")).getDeclaredMethods())[2];
    CGLIB$sayNo$2$Proxy = MethodProxy.create(clazz2, clazz1, "()V", "sayNo", "CGLIB$sayNo$2");
    ReflectUtils.findMethods(new String[] { "sayHi1", "()V", "sayHi2", "()V", "sayNo", "()V" }, (clazz2 = Class.forName("com.suncy.article.article11.cglib.Hello1")).getDeclaredMethods());
    CGLIB$equals$3$Method = ReflectUtils.findMethods(new String[] { "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;" }, (clazz2 = Class.forName("java.lang.Object")).getDeclaredMethods())[0];
    CGLIB$equals$3$Proxy = MethodProxy.create(clazz2, clazz1, "(Ljava/lang/Object;)Z", "equals", "CGLIB$equals$3");
    CGLIB$toString$4$Method = ReflectUtils.findMethods(new String[] { "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;" }, (clazz2 = Class.forName("java.lang.Object")).getDeclaredMethods())[1];
    CGLIB$toString$4$Proxy = MethodProxy.create(clazz2, clazz1, "()Ljava/lang/String;", "toString", "CGLIB$toString$4");
    CGLIB$hashCode$5$Method = ReflectUtils.findMethods(new String[] { "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;" }, (clazz2 = Class.forName("java.lang.Object")).getDeclaredMethods())[2];
    CGLIB$hashCode$5$Proxy = MethodProxy.create(clazz2, clazz1, "()I", "hashCode", "CGLIB$hashCode$5");
    CGLIB$clone$6$Method = ReflectUtils.findMethods(new String[] { "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;" }, (clazz2 = Class.forName("java.lang.Object")).getDeclaredMethods())[3];
    CGLIB$clone$6$Proxy = MethodProxy.create(clazz2, clazz1, "()Ljava/lang/Object;", "clone", "CGLIB$clone$6");
    ReflectUtils.findMethods(new String[] { "equals", "(Ljava/lang/Object;)Z", "toString", "()Ljava/lang/String;", "hashCode", "()I", "clone", "()Ljava/lang/Object;" }, (clazz2 = Class.forName("java.lang.Object")).getDeclaredMethods());
  }

  final void CGLIB$sayHi1$0() {
    super.sayHi1();
  }

  public final void sayHi1() {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    if (this.CGLIB$CALLBACK_0 != null)
      return; 
    super.sayHi1();
  }

  final void CGLIB$sayHi2$1() {
    super.sayHi2();
  }

  public final void sayHi2() {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    if (this.CGLIB$CALLBACK_0 != null)
      return; 
    super.sayHi2();
  }

  final void CGLIB$sayNo$2() {
    super.sayNo();
  }

  public final void sayNo() {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    if (this.CGLIB$CALLBACK_0 != null)
      return; 
    super.sayNo();
  }

  final boolean CGLIB$equals$3(Object paramObject) {
    return super.equals(paramObject);
  }

  public final boolean equals(Object paramObject) {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    if (this.CGLIB$CALLBACK_0 != null) {
      this.CGLIB$CALLBACK_0.intercept(this, CGLIB$equals$3$Method, new Object[] { paramObject }, CGLIB$equals$3$Proxy);
      return (this.CGLIB$CALLBACK_0.intercept(this, CGLIB$equals$3$Method, new Object[] { paramObject }, CGLIB$equals$3$Proxy) == null) ? false : ((Boolean)this.CGLIB$CALLBACK_0.intercept(this, CGLIB$equals$3$Method, new Object[] { paramObject }, CGLIB$equals$3$Proxy)).booleanValue();
    } 
    return super.equals(paramObject);
  }

  final String CGLIB$toString$4() {
    return super.toString();
  }

  public final String toString() {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    return (this.CGLIB$CALLBACK_0 != null) ? (String)this.CGLIB$CALLBACK_0.intercept(this, CGLIB$toString$4$Method, CGLIB$emptyArgs, CGLIB$toString$4$Proxy) : super.toString();
  }

  final int CGLIB$hashCode$5() {
    return super.hashCode();
  }

  public final int hashCode() {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    if (this.CGLIB$CALLBACK_0 != null) {
      this.CGLIB$CALLBACK_0.intercept(this, CGLIB$hashCode$5$Method, CGLIB$emptyArgs, CGLIB$hashCode$5$Proxy);
      return (this.CGLIB$CALLBACK_0.intercept(this, CGLIB$hashCode$5$Method, CGLIB$emptyArgs, CGLIB$hashCode$5$Proxy) == null) ? 0 : ((Number)this.CGLIB$CALLBACK_0.intercept(this, CGLIB$hashCode$5$Method, CGLIB$emptyArgs, CGLIB$hashCode$5$Proxy)).intValue();
    } 
    return super.hashCode();
  }

  final Object CGLIB$clone$6() throws CloneNotSupportedException {
    return super.clone();
  }

  protected final Object clone() throws CloneNotSupportedException {
    if (this.CGLIB$CALLBACK_0 == null)
      CGLIB$BIND_CALLBACKS(this); 
    return (this.CGLIB$CALLBACK_0 != null) ? this.CGLIB$CALLBACK_0.intercept(this, CGLIB$clone$6$Method, CGLIB$emptyArgs, CGLIB$clone$6$Proxy) : super.clone();
  }

  public static MethodProxy CGLIB$findMethodProxy(Signature paramSignature) {
    // Byte code:
    //   0: aload_0
    //   1: invokevirtual toString : ()Ljava/lang/String;
    //   4: dup
    //   5: invokevirtual hashCode : ()I
    //   8: lookupswitch default -> 160, -2007222039 -> 76, -508378822 -> 88, 1826985398 -> 100, 1913648695 -> 112, 1984935277 -> 124, 2023576048 -> 136, 2023605839 -> 148
    //   76: ldc 'sayNo()V'
    //   78: invokevirtual equals : (Ljava/lang/Object;)Z
    //   81: ifeq -> 161
    //   84: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$sayNo$2$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   87: areturn
    //   88: ldc 'clone()Ljava/lang/Object;'
    //   90: invokevirtual equals : (Ljava/lang/Object;)Z
    //   93: ifeq -> 161
    //   96: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$clone$6$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   99: areturn
    //   100: ldc 'equals(Ljava/lang/Object;)Z'
    //   102: invokevirtual equals : (Ljava/lang/Object;)Z
    //   105: ifeq -> 161
    //   108: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$equals$3$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   111: areturn
    //   112: ldc 'toString()Ljava/lang/String;'
    //   114: invokevirtual equals : (Ljava/lang/Object;)Z
    //   117: ifeq -> 161
    //   120: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$toString$4$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   123: areturn
    //   124: ldc 'hashCode()I'
    //   126: invokevirtual equals : (Ljava/lang/Object;)Z
    //   129: ifeq -> 161
    //   132: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$hashCode$5$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   135: areturn
    //   136: ldc 'sayHi1()V'
    //   138: invokevirtual equals : (Ljava/lang/Object;)Z
    //   141: ifeq -> 161
    //   144: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$sayHi1$0$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   147: areturn
    //   148: ldc 'sayHi2()V'
    //   150: invokevirtual equals : (Ljava/lang/Object;)Z
    //   153: ifeq -> 161
    //   156: getstatic com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$sayHi2$1$Proxy : Lnet/sf/cglib/proxy/MethodProxy;
    //   159: areturn
    //   160: pop
    //   161: aconst_null
    //   162: areturn
  }

  public Hello1$$EnhancerByCGLIB$$3c92030b() {
    CGLIB$BIND_CALLBACKS(this);
  }

  public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] paramArrayOfCallback) {
    CGLIB$THREAD_CALLBACKS.set(paramArrayOfCallback);
  }

  public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] paramArrayOfCallback) {
    CGLIB$STATIC_CALLBACKS = paramArrayOfCallback;
  }

  private static final void CGLIB$BIND_CALLBACKS(Object paramObject) {
    Hello1$$EnhancerByCGLIB$$3c92030b hello1$$EnhancerByCGLIB$$3c92030b = (Hello1$$EnhancerByCGLIB$$3c92030b)paramObject;
    if (!hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$BOUND) {
      hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$BOUND = true;
      if (CGLIB$THREAD_CALLBACKS.get() == null) {
        CGLIB$THREAD_CALLBACKS.get();
        if (CGLIB$STATIC_CALLBACKS == null)
          return; 
      } 
      hello1$$EnhancerByCGLIB$$3c92030b.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])CGLIB$THREAD_CALLBACKS.get())[0];
    } 
  }

  public Object newInstance(Callback[] paramArrayOfCallback) {
    CGLIB$SET_THREAD_CALLBACKS(paramArrayOfCallback);
    CGLIB$SET_THREAD_CALLBACKS(null);
    return new Hello1$$EnhancerByCGLIB$$3c92030b();
  }

  public Object newInstance(Callback paramCallback) {
    CGLIB$SET_THREAD_CALLBACKS(new Callback[] { paramCallback });
    CGLIB$SET_THREAD_CALLBACKS(null);
    return new Hello1$$EnhancerByCGLIB$$3c92030b();
  }

  public Object newInstance(Class[] paramArrayOfClass, Object[] paramArrayOfObject, Callback[] paramArrayOfCallback) {
    // Byte code:
    //   0: aload_3
    //   1: invokestatic CGLIB$SET_THREAD_CALLBACKS : ([Lnet/sf/cglib/proxy/Callback;)V
    //   4: new com/suncy/article/article11/cglib/Hello1$$EnhancerByCGLIB$$3c92030b
    //   7: dup
    //   8: aload_1
    //   9: dup
    //   10: arraylength
    //   11: tableswitch default -> 35, 0 -> 28
    //   28: pop
    //   29: invokespecial <init> : ()V
    //   32: goto -> 49
    //   35: goto -> 38
    //   38: pop
    //   39: new java/lang/IllegalArgumentException
    //   42: dup
    //   43: ldc 'Constructor not found'
    //   45: invokespecial <init> : (Ljava/lang/String;)V
    //   48: athrow
    //   49: aconst_null
    //   50: invokestatic CGLIB$SET_THREAD_CALLBACKS : ([Lnet/sf/cglib/proxy/Callback;)V
    //   53: areturn
  }

  public Callback getCallback(int paramInt) {
    CGLIB$BIND_CALLBACKS(this);
    switch (paramInt) {
      case 0:

    } 
    return null;
  }

  public void setCallback(int paramInt, Callback paramCallback) {
    switch (paramInt) {
      case 0:
        this.CGLIB$CALLBACK_0 = (MethodInterceptor)paramCallback;
        break;
    } 
  }

  public Callback[] getCallbacks() {
    CGLIB$BIND_CALLBACKS(this);
    return new Callback[] { (Callback)this.CGLIB$CALLBACK_0 };
  }

  public void setCallbacks(Callback[] paramArrayOfCallback) {
    this.CGLIB$CALLBACK_0 = (MethodInterceptor)paramArrayOfCallback[0];
  }

  static {
    CGLIB$STATICHOOK1();
  }
}
```

### 3、与jdk代理的区别

（1）jdk代理继承了Proxy，，由于java是单继承，所以只能通过接口方式进行代理。&#x20;

（2）cglib是继承了被代理的对象，重写了所有的方法。

## 四、说明

到这里，我们仅仅是知道了cglib的大概逻辑，要想清楚的了解它是怎么样生成这堆字节码文件的，还需要更深入的了解字节码的结构和ASM（java的字节码操纵框架）的用法
