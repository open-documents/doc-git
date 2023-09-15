
正常情况下，main线程的加载器是AppClassLoader，其父加载器是PlatformClassLoader，PlatformClassLoader的父加载器是BootClassLoader。


# BootClassLoader

下面是BootClassLoader的定义。
```java
private static class BootClassLoader extends BuiltinClassLoader {  
    BootClassLoader(URLClassPath bcp) {  
        super(null, null, bcp);  
    }  
  
    @Override  
    protected Class<?> loadClassOrNull(String cn, boolean resolve) {  
        return JLA.findBootstrapClassOrNull(this, cn);  
    }  
};
```

# PlatformClassLoader

下面是
```java
private static class PlatformClassLoader extends BuiltinClassLoader {  
    static {  
        if (!ClassLoader.registerAsParallelCapable())  
            throw new InternalError();  
    }  
  
    PlatformClassLoader(BootClassLoader parent) {  
        super("platform", parent, null);  
    }  
}
```
# AppClassLoader

```java
private static class AppClassLoader extends BuiltinClassLoader {  
    static {  
        if (!ClassLoader.registerAsParallelCapable())  
            throw new InternalError();  
    }  
  
    AppClassLoader(BuiltinClassLoader parent, URLClassPath ucp) {  
        super("app", parent, ucp);  
    }  
  
    @Override  
    protected Class<?> loadClass(String cn, boolean resolve) throws ClassNotFoundException{  
        // for compatibility reasons, say where restricted package list has  
        // been updated to list API packages in the unnamed module.        
        SecurityManager sm = System.getSecurityManager();  
        if (sm != null) {  
            int i = cn.lastIndexOf('.');  
            if (i != -1) {  
                sm.checkPackageAccess(cn.substring(0, i));  
            }  
        }  
  
        return super.loadClass(cn, resolve);  
    }  
  
    @Override  
    protected PermissionCollection getPermissions(CodeSource cs) {  
        PermissionCollection perms = super.getPermissions(cs);  
        perms.add(new RuntimePermission("exitVM"));  
        return perms;  
    }  
  
    /**  
     * Called by the VM to support dynamic additions to the class path     *     * @see java.lang.instrument.Instrumentation#appendToSystemClassLoaderSearch  
     */    void appendToClassPathForInstrumentation(String path) {  
        appendClassPath(path);  
    }  
  
    /**  
     * Called by the VM to support define package for AppCDS     */    protected Package defineOrCheckPackage(String pn, Manifest man, URL url) {  
        return super.defineOrCheckPackage(pn, man, url);  
    }  
  
    /**  
     * Called by the VM, during -Xshare:dump     */    private void resetArchivedStates() {  
        setClassPath(null);  
    }  
}
```