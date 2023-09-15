# 准备工作

**1）需要被增强的类**
```java
public class BaseBean {  
    public void say() {  
        System.out.println("say()...");  
    }  
}
```

**2）拦截器**
```java
public class CustomizedMethodInterceptor implements MethodInterceptor {  
    @Override  
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) 
throws Throwable {  
        System.out.println("invoke " + method.getName() + " before ! ");  
        Object result = methodProxy.invokeSuper(o, objects);  
        System.out.println("invoke " + method.getName() + " after ! ");  
        return result;  
    }
}
```



# 原理解析

```java
public class BaseBean$$EnhancerByCGLIB$$5b0d0269 extends BaseBean implements Factory {  
    private boolean CGLIB$BOUND;  
    public static Object CGLIB$FACTORY_DATA;  
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;  
    private static final Callback[] CGLIB$STATIC_CALLBACKS;  
    private MethodInterceptor CGLIB$CALLBACK_0;  
    private static Object CGLIB$CALLBACK_FILTER;  
    private static final Method CGLIB$say$0$Method;  
    private static final MethodProxy CGLIB$say$0$Proxy;  
    private static final Object[] CGLIB$emptyArgs;  
    ...  // 与Object类方法相关
    
    
    static void CGLIB$STATICHOOK1() {  
        CGLIB$THREAD_CALLBACKS = new ThreadLocal();  
        CGLIB$emptyArgs = new Object[0];  
        Class var0 = Class.forName("org.example.BaseBean$$EnhancerByCGLIB$$5b0d0269");  
        Class var1;  
        ...  // 与Object类方法相关
        CGLIB$say$0$Method = ReflectUtils.findMethods(new String[]{"say", "()V"}, (var1 = Class.forName("org.example.BaseBean")).getDeclaredMethods())[0];  
        CGLIB$say$0$Proxy = MethodProxy.create(var1, var0, "()V", "say", "CGLIB$say$0");  
    }

    final void CGLIB$say$0() {  
        super.say();  

    public final void say() {  
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;  
        if (var10000 == null) {  
            CGLIB$BIND_CALLBACKS(this);  
            var10000 = this.CGLIB$CALLBACK_0;  
        }  
        if (var10000 != null) {
            var10000.intercept(this, CGLIB$say$0$Method, CGLIB$emptyArgs, CGLIB$say$0$Proxy);  
        } else {  
            super.say();  
        }  
    }  

    ...  // 与Object类方法相关

    private static final void CGLIB$BIND_CALLBACKS(Object var0) {  
        BaseBean$$EnhancerByCGLIB$$5b0d0269 var1 = (BaseBean$$EnhancerByCGLIB$$5b0d0269)var0;  
        if (!var1.CGLIB$BOUND) {  
            var1.CGLIB$BOUND = true;  
            Object var10000 = CGLIB$THREAD_CALLBACKS.get();  
            if (var10000 == null) {  
                var10000 = CGLIB$STATIC_CALLBACKS;  
                if (var10000 == null) {  
                    return;  
                }  
            }  
            var1.CGLIB$CALLBACK_0 = (MethodInterceptor)((Callback[])var10000)[0];  
        }  
    }  
    
    public static MethodProxy CGLIB$findMethodProxy(Signature var0) {  
        String var10000 = var0.toString();  
        switch(var10000.hashCode()) {  
        case -909388886:  
            if (var10000.equals("say()V")) {  
                return CGLIB$say$0$Proxy;  
            }  
            break;  
        ...  // // 与Object类方法相关
        return null;  
    }  

    public BaseBean$$EnhancerByCGLIB$$5b0d0269() {  
        CGLIB$BIND_CALLBACKS(this);  
    }

    public static void CGLIB$SET_THREAD_CALLBACKS(Callback[] var0) {  
        CGLIB$THREAD_CALLBACKS.set(var0);  
    }  
  
    public static void CGLIB$SET_STATIC_CALLBACKS(Callback[] var0) {  
        CGLIB$STATIC_CALLBACKS = var0;  
    }  

    public Object newInstance(Callback[] var1) {  
        CGLIB$SET_THREAD_CALLBACKS(var1);  
        BaseBean$$EnhancerByCGLIB$$5b0d0269 var10000 = new BaseBean$$EnhancerByCGLIB$$5b0d0269();  
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);  
        return var10000;  
    }  
  
    public Object newInstance(Callback var1) {  
        CGLIB$SET_THREAD_CALLBACKS(new Callback[]{var1});  
        BaseBean$$EnhancerByCGLIB$$5b0d0269 var10000 = new BaseBean$$EnhancerByCGLIB$$5b0d0269();  
        CGLIB$SET_THREAD_CALLBACKS((Callback[])null);  
        return var10000;  
    }  
  
    public Object newInstance(Class[] var1, Object[] var2, Callback[] var3) {  
        CGLIB$SET_THREAD_CALLBACKS(var3);  
        BaseBean$$EnhancerByCGLIB$$5b0d0269 var10000 = new BaseBean$$EnhancerByCGLIB$$5b0d0269;  
        switch(var1.length) {  
        case 0:  
            var10000.<init>();  
            CGLIB$SET_THREAD_CALLBACKS((Callback[])null);  
            return var10000;  
        default:  
            throw new IllegalArgumentException("Constructor not found");  
        }  
    }  
  
    public Callback getCallback(int var1) {  
        CGLIB$BIND_CALLBACKS(this);  
        MethodInterceptor var10000;  
        switch(var1) {  
        case 0:  
            var10000 = this.CGLIB$CALLBACK_0;  
            break;  
        default:  
            var10000 = null;  
        }  
  
        return var10000;  
    }  
  
    public void setCallback(int var1, Callback var2) {  
        switch(var1) {  
        case 0:  
            this.CGLIB$CALLBACK_0 = (MethodInterceptor)var2;  
        default:  
        }  
    }  
  
    public Callback[] getCallbacks() {  
        CGLIB$BIND_CALLBACKS(this);  
        return new Callback[]{this.CGLIB$CALLBACK_0};  
    }  
  
    public void setCallbacks(Callback[] var1) {  
        this.CGLIB$CALLBACK_0 = (MethodInterceptor)var1[0];  
    }  
  
    static {  
        CGLIB$STATICHOOK1();  
    }  
}
 
 
```