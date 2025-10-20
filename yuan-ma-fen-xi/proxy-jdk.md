# jdk动态代理源码分析

## 一、用法

### 1、接口

```
package com.suncy.article.article11.jdk;

public interface HelloInterface {
    public void sayHi();
}
```

### 2、接口实现类（要代理的对象就是这个类的对象）

```
package com.suncy.article.article11;

import com.suncy.article.article11.jdk.HelloInterface;

public class Hello implements HelloInterface {
    @Override
    public void sayHi() {
        System.out.println("hi");
    }
}
```

### 3、执行代理对象方法的前后完成自己的操作

```
package com.suncy.article.article11.jdk;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class JdkProxy implements InvocationHandler {

    private Object object;

    public JdkProxy(Object object){
        this.object = object;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        System.out.println("jdk before");
        method.invoke(object,args);
        System.out.println("jdk after");
        return null;
    }
}
```

### 4、使用

```
package com.suncy.article.article11.jdk;

import com.suncy.article.article11.Hello;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class JdkTest {
    public static void main(String[] args) {
        //1、生成一个普通对象（必须要实现接口）
        Hello hello = new Hello();

        //2、代理对象，手动编写代理方法的前后的操作（这个类模型比较固定）
        InvocationHandler invocationHandler = new JdkProxy(hello);

        //3、使用反射包中的Proxy.newProxyInstance方法，拿到代理对象（参数为 被代理对象的类加载器 接口列表Class<?> InvocationHandler）
        HelloInterface helloInterface = (HelloInterface) Proxy.newProxyInstance(
                hello.getClass().getClassLoader(),
                hello.getClass().getInterfaces(),
                invocationHandler
        );

        //4、调用代理对象的接口方法
        helloInterface.sayHi();
    }
}
```

### 5、测试结果

![测试结果](<../.gitbook/assets/image (31).png>)

## 二、源码分析

### **（1）Proxy.newProxyInstance()方法**&#x20;

Proxy.newProxyInstance()拿到代理对象

```
HelloInterface helloInterface = (HelloInterface) Proxy.newProxyInstance(
                hello.getClass().getClassLoader(),
                hello.getClass().getInterfaces(),
                invocationHandler
        );
```

### **（2）分析这个方法**

```
public static Object newProxyInstance(ClassLoader loader,
                                      Class<?>[] interfaces,
                                      InvocationHandler h)
    throws IllegalArgumentException
{
    Objects.requireNonNull(h);

    //1、拿到接口的Class[] 数组
    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }

    //2、拿到代理类的Class<?>对象
    Class<?> cl = getProxyClass0(loader, intfs);

    try {
        if (sm != null) {
            checkNewProxyPermission(Reflection.getCallerClass(), cl);
        }

        //3、通过Class<?> 拿到Constructor对象
        final Constructor<?> cons = cl.getConstructor(constructorParams);
        final InvocationHandler ih = h;
        if (!Modifier.isPublic(cl.getModifiers())) {
            AccessController.doPrivileged(new PrivilegedAction<Void>() {
                public Void run() {
                    cons.setAccessible(true);
                    return null;
                }
            });
        }
        //4、通过Constructor返回代理对象
        return cons.newInstance(new Object[]{h});
    } catch (IllegalAccessException|InstantiationException e) {
        throw new InternalError(e.toString(), e);
    } catch (InvocationTargetException e) {
        Throwable t = e.getCause();
        if (t instanceof RuntimeException) {
            throw (RuntimeException) t;
        } else {
            throw new InternalError(t.toString(), t);
        }
    } catch (NoSuchMethodException e) {
        throw new InternalError(e.toString(), e);
    }
}
```

首先看返回值

```
 return cons.newInstance(new Object[]{h});
```

然后找到cons

```
 final Constructor<?> cons = cl.getConstructor(constructorParams);
```

继续找到cl，发现通过getProxyClass0方法 返回了一个Class\<?> 对象 ，这个getProxyClass0方法的参数loader和intfs就是传进来的参数

```
    Class<?> cl = getProxyClass0(loader, intfs);
```

到这里，我们应该知道关键的方法就是在于getProxyClass0，拿到一个类的Class对象，之后通过这个Class对象反射出代理对象。

### **（3）找到getProxyClass0()**

```
    private static Class<?> getProxyClass0(ClassLoader loader,
                                           Class<?>... interfaces) {
        if (interfaces.length > 65535) {
            throw new IllegalArgumentException("interface limit exceeded");
        }

        return proxyClassCache.get(loader, interfaces);
    }
```

直接return proxyClassCache.get(loader, interfaces)。

### **（4）找到proxyClassCache.get()**

```
public V get(K key, P parameter) {
    Objects.requireNonNull(parameter);

    expungeStaleEntries();

    Object cacheKey = CacheKey.valueOf(key, refQueue);

    ConcurrentMap<Object, Supplier<V>> valuesMap = map.get(cacheKey);
    if (valuesMap == null) {
        ConcurrentMap<Object, Supplier<V>> oldValuesMap
            = map.putIfAbsent(cacheKey,
                              valuesMap = new ConcurrentHashMap<>());
        if (oldValuesMap != null) {
            valuesMap = oldValuesMap;
        }
    }

    Object subKey = Objects.requireNonNull(subKeyFactory.apply(key, parameter));
    Supplier<V> supplier = valuesMap.get(subKey);
    Factory factory = null;

    while (true) {
        //第一次进来supplier为null
        //第二次进来supplier不为null ， 
        if (supplier != null) {
            V value = supplier.get();
            if (value != null) {
                return value;
            }
        }

        //第一次循环，factort为null，此时会新建new Factory()
        if (factory == null) {
            factory = new Factory(key, parameter, subKey, valuesMap);
        }

        if (supplier == null) {
            supplier = valuesMap.putIfAbsent(subKey, factory);
            if (supplier == null) {
                //第一次循环会走到这个地方，将这个factory赋值给supplier
                supplier = factory;
            }
        } else {
            if (valuesMap.replace(subKey, supplier, factory)) {
                supplier = factory;
            } else {
                supplier = valuesMap.get(subKey);
            }
        }
    }
}
```

先看返回值，在while(true)代码块中

```
if (supplier != null) {
       // supplier might be a Factory or a CacheValue<V> instance
        V value = supplier.get();
        if (value != null) {
              return value;
        }
 }
```

那就需要找到这个supplier是什么？

断点测试如下： 第一次循环&#x20;

![第一次循环](<../.gitbook/assets/image (17).png>)

第二次循环&#x20;

![第二次循环](<../.gitbook/assets/image (33).png>)

最后我们找到这个supplier,get() 调用的就是WeakCache类下的一个内部类Factory的get方法

### **（5）分析Factory的get方法**

```
public synchronized V get() { // serialize access
        Supplier<V> supplier = valuesMap.get(subKey);
        if (supplier != this) {
            return null;
        }

        V value = null;
        try {
            value = Objects.requireNonNull(valueFactory.apply(key, parameter));
        } finally {
            if (value == null) { // remove us on failure
                valuesMap.remove(subKey, this);
            }
        }
        assert value != null;

        CacheValue<V> cacheValue = new CacheValue<>(value);

        if (valuesMap.replace(subKey, this, cacheValue)) {
            reverseMap.put(cacheValue, Boolean.TRUE);
        } else {
            throw new AssertionError("Should not reach here");
        }

        return value;
    }
}
```

依旧去找返回值，简略如下：

```
value = Objects.requireNonNull(valueFactory.apply(key, parameter));
return value;
```

Objects.requireNonNull就是为了返回参数对象，参数对象为null，则抛出异常，方法不关键。 关键方法在于valueFactory.apply(key, parameter)。

### **（6）找到valueFactory.apply(key, parameter)方法**

&#x20;通过断点找到的是ProxyClassFactory类下的apply方法

```
 @Override
    public Class<?> apply(ClassLoader loader, Class<?>[] interfaces) {

        Map<Class<?>, Boolean> interfaceSet = new IdentityHashMap<>(interfaces.length);
        for (Class<?> intf : interfaces) {

            Class<?> interfaceClass = null;
            try {
                interfaceClass = Class.forName(intf.getName(), false, loader);
            } catch (ClassNotFoundException e) {
            }
            if (interfaceClass != intf) {
                throw new IllegalArgumentException(
                    intf + " is not visible from class loader");
            }

            if (!interfaceClass.isInterface()) {
                throw new IllegalArgumentException(
                    interfaceClass.getName() + " is not an interface");
            }

            if (interfaceSet.put(interfaceClass, Boolean.TRUE) != null) {
                throw new IllegalArgumentException(
                    "repeated interface: " + interfaceClass.getName());
            }
        }

        String proxyPkg = null;     // package to define proxy class in
        int accessFlags = Modifier.PUBLIC | Modifier.FINAL;

        for (Class<?> intf : interfaces) {
            int flags = intf.getModifiers();
            if (!Modifier.isPublic(flags)) {
                accessFlags = Modifier.FINAL;
                String name = intf.getName();
                int n = name.lastIndexOf('.');
                String pkg = ((n == -1) ? "" : name.substring(0, n + 1));
                if (proxyPkg == null) {
                    proxyPkg = pkg;
                } else if (!pkg.equals(proxyPkg)) {
                    throw new IllegalArgumentException(
                        "non-public interfaces from different packages");
                }
            }
        }

        if (proxyPkg == null) {
            proxyPkg = ReflectUtil.PROXY_PACKAGE + ".";
        }

        long num = nextUniqueNumber.getAndIncrement();
        String proxyName = proxyPkg + proxyClassNamePrefix + num;

        //返回值要用到的参数 byte[] proxyClassFile
        byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
        try {
            //返回值，关键地方
            return defineClass0(loader, proxyName,
                                proxyClassFile, 0, proxyClassFile.length);
        } catch (ClassFormatError e) {
            throw new IllegalArgumentException(e.toString());
        }
    }
}
```

这时候我们发现这个方法的参数返回了一个 Class\<?>对象，而且入参就是类加载器和接口 Class\<?>数组。是不是刚好对应了Proxy.newProxyInstance这个方法的两个入参和出参。那么这个方法一定非常关键了。

### **（7）defineClass0方法** 依旧去看apply方法的返回值：

```
  return defineClass0(loader, proxyName, proxyClassFile, 0, proxyClassFile.length);
```

然后我们点击进入这个defineClass0方法发现，这是一个native方法，返回Class\<?>对象，然后我们找一下资料，看一下这个方法的作用。

```
private static native Class<?> defineClass0(ClassLoader loader, String name,
                                                byte[] b, int off, int len);
```

通过查找资料，我们发现：这个defineClass0方法位于java的反射包当中，作用就是将一堆字节转化为Class\<?>对象。知道了这个，那我们就知道，关键的地方应该在于这堆字节是怎么来的？

### **（8）ProxyGenerator.generateProxyClas方法**

```
 byte[] proxyClassFile = ProxyGenerator.generateProxyClass(
            proxyName, interfaces, accessFlags);
```

进入到generateProxyClass方法当中

```
public static byte[] generateProxyClass(final String var0, Class<?>[] var1, int var2) {
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    final byte[] var4 = var3.generateClassFile();
    if (saveGeneratedFiles) {
        AccessController.doPrivileged(new PrivilegedAction<Void>() {
            public Void run() {
                try {
                    int var1 = var0.lastIndexOf(46);
                    Path var2;
                    if (var1 > 0) {
                        Path var3 = Paths.get(var0.substring(0, var1).replace('.', File.separatorChar));
                        Files.createDirectories(var3);
                        var2 = var3.resolve(var0.substring(var1 + 1, var0.length()) + ".class");
                    } else {
                        var2 = Paths.get(var0 + ".class");
                    }

                    Files.write(var2, var4, new OpenOption[0]);
                    return null;
                } catch (IOException var4x) {
                    throw new InternalError("I/O exception saving generated file: " + var4x);
                }
            }
        });
    }

    return var4;
}
```

去找返回值，简略如下：

```
    ProxyGenerator var3 = new ProxyGenerator(var0, var1, var2);
    final byte[] var4 = var3.generateClassFile();
    return var4;
```

### **（9）ProxyGenerator的generateClassFile方法**

```
private byte[] generateClassFile() {
    //为代理类生成hashCode、equales、toString方法
    this.addProxyMethod(hashCodeMethod, Object.class);
    this.addProxyMethod(equalsMethod, Object.class);
    this.addProxyMethod(toStringMethod, Object.class);
    Class[] var1 = this.interfaces;
    int var2 = var1.length;

    int var3;
    Class var4;
    //遍历每一个接口的每一个方法, 并且为其生成ProxyMethod对象
    for(var3 = 0; var3 < var2; ++var3) {
        var4 = var1[var3];
        Method[] var5 = var4.getMethods();
        int var6 = var5.length;

        for(int var7 = 0; var7 < var6; ++var7) {
            Method var8 = var5[var7];
            this.addProxyMethod(var8, var4);
        }
    }

    Iterator var11 = this.proxyMethods.values().iterator();

    List var12;
    while(var11.hasNext()) {
        var12 = (List)var11.next();
        checkReturnTypes(var12);
    }

    Iterator var15;
    try {
        //添加构造器方法
        this.methods.add(this.generateConstructor());
        var11 = this.proxyMethods.values().iterator();

        while(var11.hasNext()) {
            var12 = (List)var11.next();
            var15 = var12.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.ProxyMethod var16 = (ProxyGenerator.ProxyMethod)var15.next();
                this.fields.add(new ProxyGenerator.FieldInfo(var16.methodFieldName, "Ljava/lang/reflect/Method;", 10));
                this.methods.add(var16.generateMethod());
            }
        }

        this.methods.add(this.generateStaticInitializer());
    } catch (IOException var10) {
        throw new InternalError("unexpected I/O Exception", var10);
    }

    //方法和字段集合不能大于65535
    if (this.methods.size() > 65535) {
        throw new IllegalArgumentException("method limit exceeded");
    } else if (this.fields.size() > 65535) {
        throw new IllegalArgumentException("field limit exceeded");
    } else {
        this.cp.getClass(dotToSlash(this.className));
        this.cp.getClass("java/lang/reflect/Proxy");
        var1 = this.interfaces;
        var2 = var1.length;

        for(var3 = 0; var3 < var2; ++var3) {
            var4 = var1[var3];
            this.cp.getClass(dotToSlash(var4.getName()));
        }

        this.cp.setReadOnly();
        ByteArrayOutputStream var13 = new ByteArrayOutputStream();
        DataOutputStream var14 = new DataOutputStream(var13);

        try {
            //这个地方，按照Class文件结构进行动态拼接的
            var14.writeInt(-889275714);
            var14.writeShort(0);
            var14.writeShort(49);
            this.cp.write(var14);
            var14.writeShort(this.accessFlags);
            var14.writeShort(this.cp.getClass(dotToSlash(this.className)));
            var14.writeShort(this.cp.getClass("java/lang/reflect/Proxy"));
            var14.writeShort(this.interfaces.length);
            Class[] var17 = this.interfaces;
            int var18 = var17.length;

            for(int var19 = 0; var19 < var18; ++var19) {
                Class var22 = var17[var19];
                var14.writeShort(this.cp.getClass(dotToSlash(var22.getName())));
            }

            var14.writeShort(this.fields.size());
            var15 = this.fields.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.FieldInfo var20 = (ProxyGenerator.FieldInfo)var15.next();
                var20.write(var14);
            }

            var14.writeShort(this.methods.size());
            var15 = this.methods.iterator();

            while(var15.hasNext()) {
                ProxyGenerator.MethodInfo var21 = (ProxyGenerator.MethodInfo)var15.next();
                var21.write(var14);
            }

            var14.writeShort(0);
            return var13.toByteArray();
        } catch (IOException var9) {
            throw new InternalError("unexpected I/O Exception", var9);
        }
    }
}
```

### （10）流程总结

1、最终通过generateClassFile方法生成了能够描述类详细信息的一堆字节。 2、通过defineClass0方法，根据字节生成了Class\<?>对象。 3、通过这个Class\<?>对象，实例化了代理对象。 4、调用代理对象的接口方法。

## 三、查看代理对象的结构

### 1、生成代理Class文件

```
package com.suncy.article.article11.jdk;

import com.suncy.article.article11.Hello;
import sun.misc.ProxyGenerator;

import java.io.FileOutputStream;
import java.io.IOException;

public class JdkProxyTest {
    public static void main(String[] args) {

        byte[] bytes = ProxyGenerator.generateProxyClass("$Proxy0", new Class[]{Hello.class});

        try {
            FileOutputStream fileOutputStream = new FileOutputStream("D:\\work\\tool\\$Proxy0.class");
            fileOutputStream.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

### 2、查看反编译后的Class代码

下载并安装JD-GUI工具、打开，并将$Proxy0.class拖入即可

```
import com.suncy.article.article11.Hello;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.UndeclaredThrowableException;

public final class $Proxy0 extends Proxy implements Hello {
  private static Method m1;

  private static Method m3;

  private static Method m8;

  private static Method m2;

  private static Method m5;

  private static Method m4;

  private static Method m7;

  private static Method m9;

  private static Method m0;

  private static Method m6;

  public $Proxy0(InvocationHandler paramInvocationHandler) {
    super(paramInvocationHandler);
  }

  public final boolean equals(Object paramObject) {
    try {
      return ((Boolean)this.h.invoke(this, m1, new Object[] { paramObject })).booleanValue();
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void sayHi() {
    try {
      this.h.invoke(this, m3, null);
      return;
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void notify() {
    try {
      this.h.invoke(this, m8, null);
      return;
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final String toString() {
    try {
      return (String)this.h.invoke(this, m2, null);
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void wait(long paramLong) throws InterruptedException {
    try {
      this.h.invoke(this, m5, new Object[] { Long.valueOf(paramLong) });
      return;
    } catch (Error|RuntimeException|InterruptedException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void wait(long paramLong, int paramInt) throws InterruptedException {
    try {
      this.h.invoke(this, m4, new Object[] { Long.valueOf(paramLong), Integer.valueOf(paramInt) });
      return;
    } catch (Error|RuntimeException|InterruptedException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final Class getClass() {
    try {
      return (Class)this.h.invoke(this, m7, null);
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void notifyAll() {
    try {
      this.h.invoke(this, m9, null);
      return;
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final int hashCode() {
    try {
      return ((Integer)this.h.invoke(this, m0, null)).intValue();
    } catch (Error|RuntimeException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  public final void wait() throws InterruptedException {
    try {
      this.h.invoke(this, m6, null);
      return;
    } catch (Error|RuntimeException|InterruptedException error) {
      throw null;
    } catch (Throwable throwable) {
      throw new UndeclaredThrowableException(throwable);
    } 
  }

  static {
    try {
      m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] { Class.forName("java.lang.Object") });
      m3 = Class.forName("com.suncy.article.article11.Hello").getMethod("sayHi", new Class[0]);
      m8 = Class.forName("com.suncy.article.article11.Hello").getMethod("notify", new Class[0]);
      m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
      m5 = Class.forName("com.suncy.article.article11.Hello").getMethod("wait", new Class[] { long.class });
      m4 = Class.forName("com.suncy.article.article11.Hello").getMethod("wait", new Class[] { long.class, int.class });
      m7 = Class.forName("com.suncy.article.article11.Hello").getMethod("getClass", new Class[0]);
      m9 = Class.forName("com.suncy.article.article11.Hello").getMethod("notifyAll", new Class[0]);
      m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
      m6 = Class.forName("com.suncy.article.article11.Hello").getMethod("wait", new Class[0]);
      return;
    } catch (NoSuchMethodException noSuchMethodException) {
      throw new NoSuchMethodError(noSuchMethodException.getMessage());
    } catch (ClassNotFoundException classNotFoundException) {
      throw new NoClassDefFoundError(classNotFoundException.getMessage());
    } 
  }
}
```

我们发现动态代理类继承了Proxy类，同时调用方法采用的是 this.h.invoke(this, m3, null);

h涞源：

```
  public $Proxy0(InvocationHandler paramInvocationHandler) {
    super(paramInvocationHandler);
  }
```

父类Proxy的构造函数，将InvocationHandler实例传进来

```
    protected Proxy(InvocationHandler h) {
        Objects.requireNonNull(h);
        this.h = h;
    }
```

m3涞源：

```
m3 = Class.forName("com.suncy.article.article11.Hello").getMethod("sayHi", new Class[0]);
```

## 三、总结

到此，我们基本上已经理清了jdk动态代理的流程，同样，我们也能拿到经过代理之后的Class对象。详细的东西可以再去分析一下这个Class对象。
