
# 1.1、getResources(String name)

name是用 '/' 分割的标志资源的路径。

```java
public Enumeration<URL> getResources(String name) throws IOException {  
    Enumeration<URL>[] tmp = (Enumeration<URL>[]) new Enumeration<?>[2];  
    if (parent != null) {
        tmp[0] = parent.getResources(name);  
    } else {  
	    // return ClassLoaders.bootLoader().findResources(name);
        tmp[0] = BootLoader.findResources(name);  
    }  
    tmp[1] = findResources(name);  

    return new CompoundEnumeration<>(tmp);  
}
```

对于main线程的类加载器AppClassLoader来说，getResources(String name)获取的CompoundEnumeration是下面这样，由此可见，真正获取资源的方法还是得看findResources()。
![[Pasted image 20230528195822.png]]
```java
Enumeration<URL> urls = xxx.class.getClassLoader().getResources(...)  // CompoundEnumeration
while (urls.hasMoreElements()) {  
   URL url = urls.nextElement();  
   ...
}
```



# 1.2、findResources()

```java
// ------------------------------------- BuiltinClassLoader -----------------------------------------------------
public Enumeration<URL> findResources(String name) throws IOException {  
    List<URL> checked = new ArrayList<>();  // list of checked URLs  

	// 转换逻辑:
	// 如果name不包含'/' 或 '/'是最后一个字符,返回null
	// 截取name到最后一个'/'的位置,然后将其中的'/'转化为'.'
	// 如 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
	// 返回META-INF.spring
    String pn = Resources.toPackageName(name);  
    LoadedModule module = packageToModule.get(pn);  
    if (module != null) {  
        // resource is in a package of a module defined to this loader  
        if (module.loader() == this) {  
            URL url = findResource(module.name(), name); // checks URL  
            if (url != null  
                && (name.endsWith(".class")  
                    || url.toString().endsWith("/")  
                    || isOpen(module.mref(), pn))) {  
                checked.add(url);  
            }  
        }  
    } else {  // 这个包不在该ClassLoader加载的模组中
        for (URL url : findMiscResource(name)) {  
            url = checkURL(url);  
            if (url != null) {  
                checked.add(url);  
            }  
        }  
    }  
    // class path (not checked)  
    // 主要调用URLClassPath.findResources(name,false)
    /*
	 *	return new Enumeration<>() {  
	 *	    private int index = 0;  
	 *	    private URL url = null;  
	 *	  
	 *	    private boolean next() {  
	 *	        if (url != null) {  
	 *	            return true;  
	 *	        } else {  
	 *	            Loader loader;  
	 *	            while ((loader = getLoader(index++)) != null) {  
	 *	                url = loader.findResource(name, check);  
	 *	                if (url != null) {  
	 *	                    return true;  
	 *	                }  
	 *	            }  
	 *	            return false;  
	 *	        }  
	 *	    }  
	 *	  
	 *	    public boolean hasMoreElements() {  
	 *	        return next();  
	 *	    }  
	 *	  
	 *	    public URL nextElement() {  
	 *	        if (!next()) {  
	 *	            throw new NoSuchElementException();  
	 *	        }  
	 *	        URL u = url;  
	 *	        url = null;  
	 *	        return u;  
	 *	    }  
	 *	};
	*/
    Enumeration<URL> e = findResourcesOnClassPath(name);  

    // concat the checked URLs and the (not checked) class path  
    return new Enumeration<>() {  
        final Iterator<URL> iterator = checked.iterator();  
        URL next;  
        private boolean hasNext() {  
            if (next != null) {  
                return true;  
            } else if (iterator.hasNext()) {  
                next = iterator.next();  
                return true;
            } else {  
                // need to check each URL  
                while (e.hasMoreElements() && next == null) {  
                    next = checkURL(e.nextElement());  
                }  
                return next != null;  
            }  
        }  
        @Override  
        public boolean hasMoreElements() {  
            return hasNext();  
        }  
        @Override  
        public URL nextElement() {  
            if (hasNext()) {  
                URL result = next;  
                next = null;  
                return result;  
            } else {...}  
        }  
    };  
  
}
```
## findMiscResource()
```java
// Misc : miscellaneous
private List<URL> findMiscResource(String name) throws IOException {  
    SoftReference<Map<String, List<URL>>> ref = this.resourceCache;  
    Map<String, List<URL>> map = (ref != null) ? ref.get() : null;  
    if (map == null) {  
        // only cache resources after VM is fully initialized  
        if (VM.isModuleSystemInited()) {  
            map = new ConcurrentHashMap<>();  
            this.resourceCache = new SoftReference<>(map);  
        }  
    } else {  
        List<URL> urls = map.get(name);  
        if (urls != null)  
            return urls;  
    }  
    // search all modules for the resource  
    List<URL> urls;  
    try {  
        urls = AccessController.doPrivileged(  
            new PrivilegedExceptionAction<>() {  
                @Override  
                public List<URL> run() throws IOException {  
                    List<URL> result = null;  
                    // nameToModule : Map<String, ModuleReference>
                    for (ModuleReference mref : nameToModule.values()) {  
                        URI u = moduleReaderFor(mref).find(name).orElse(null);  
                        if (u != null) {  
                            try {  
                                if (result == null)  
                                    result = new ArrayList<>();  
                                result.add(u.toURL());  
                            } catch (MalformedURLException |  
                                     IllegalArgumentException e) {  
                            }  
                        }  
                    }  
                    return (result != null) ? result : Collections.emptyList();  
                }  
            });  
    } catch (PrivilegedActionException pae) {  ...  }  

    // only cache resources after VM is fully initialized  
    if (map != null) {  
        map.putIfAbsent(name, urls);  
    }
  
    return urls;  
}
```
## findResourcesOnClassPath()
```java
private Enumeration<URL> findResourcesOnClassPath(String name) {  
	// return ucp != null;
    if (hasClassPath()) {  
        if (System.getSecurityManager() == null) {  
            return ucp.findResources(name, false);  
        } else {  
            PrivilegedAction<Enumeration<URL>> pa;  
            pa = () -> ucp.findResources(name, false);  
            return AccessController.doPrivileged(pa);  
        }  
    } else {  
        // no class path  
        return Collections.emptyEnumeration();  
    }  
}
```

## URLClassPath.getLoader(int)

```java
private synchronized Loader getLoader(int index) {  
    if (closed) {  
        return null;  
    }  
    // Expand URL search path until the request can be satisfied  
    // or unopenedUrls is exhausted.    
    while (loaders.size() < index + 1) {  // private final ArrayList<Loader> loaders = new ArrayList<>();
        final URL url;  
        synchronized (unopenedUrls) {  
            url = unopenedUrls.pollFirst();  
            if (url == null)  
                return null;  
        }  
        // Skip this URL if it already has a Loader. (Loader  
        // may be null in the case where URL has not been opened        // but is referenced by a JAR index.)        String urlNoFragString = URLUtil.urlNoFragString(url);  
        if (lmap.containsKey(urlNoFragString)) {  
            continue;  
        }  
        // Otherwise, create a new Loader for the URL.  
        Loader loader;  
        try {  
            loader = getLoader(url);  
            // If the loader defines a local class path then add the  
            // URLs as the next URLs to be opened.            URL[] urls = loader.getClassPath();  
            if (urls != null) {  
                push(urls);  
            }  
        } catch (IOException e) {  
            // Silently ignore for now...  
            continue;  
        } catch (SecurityException se) {  
            // Always silently ignore. The context, if there is one, that  
            // this URLClassPath was given during construction will never            // have permission to access the URL. 
            if (DEBUG) {  
                System.err.println("Failed to access " + url + ", " + se );  
            }  
            continue;  
        }  
        // Finally, add the Loader to the search path.  
        loaders.add(loader);  
        lmap.put(urlNoFragString, loader);  
    }  
    return loaders.get(index);  
}
```
## 



# checkURL()

```java
private static URL checkURL(URL url) {  
    return URLClassPath.checkURL(url);  
}
```

# ClassLoader$CompoundEnumeration

下面是该类的定义。
```java
final class CompoundEnumeration<E> implements Enumeration<E> {  
    private final Enumeration<E>[] enums;  
    private int index;  
  
    public CompoundEnumeration(Enumeration<E>[] enums) {  
        this.enums = enums;  
    }  
  
    private boolean next() {  
        while (index < enums.length) {  
            if (enums[index] != null && enums[index].hasMoreElements()) {  
                return true;  
            }  
            index++;  
        }  
        return false;  
    }  
  
    public boolean hasMoreElements() {  
        return next();  
    }  
  
    public E nextElement() {  
        if (!next()) {...}  
        return enums[index].nextElement();  
    }  
}
```
针对下面的代码（一般这样使用CompoundEnumertaion）以及结构，作逻辑说明。
![[Pasted image 20230528195822.png]]
```java
Enumeration<URL> urls = xxx.class.getClassLoader().getResources(...)  // CompoundEnumeration
while (urls.hasMoreElements()) {  
   URL url = urls.nextElement();  
   ...
}
```
