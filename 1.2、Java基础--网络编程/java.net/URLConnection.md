


# 缓存

```java
private static final ConcurrentHashMap<String,Boolean> defaultCaching = new ConcurrentHashMap<>();

public static void setDefaultUseCaches(String protocol, boolean defaultVal) {  
    protocol = protocol.toLowerCase(Locale.US);  
    defaultCaching.put(protocol, defaultVal);  
}
```

```java
private static volatile boolean defaultUseCaches = true;

protected boolean useCaches;
public void setUseCaches(boolean usecaches) {  
    checkConnected();  
    useCaches = usecaches;  
}
public boolean getUseCaches() {  
    return useCaches;  
}
```

