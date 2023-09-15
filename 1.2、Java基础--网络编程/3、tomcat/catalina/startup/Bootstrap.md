```java
public final class Bootstrap{
	static {  
	    // Will always be non-null  
	    String userDir = System.getProperty("user.dir");  
	  
	    // Home first  
	    String home = System.getProperty(Constants.CATALINA_HOME_PROP);  // "catalina.home"
	    File homeFile = null;  
	  
	    if (home != null) {  
	        File f = new File(home);  
	        try {  
	            homeFile = f.getCanonicalFile();  
	        } catch (IOException ioe) {  
	            homeFile = f.getAbsoluteFile();  
	        }  
	    }  
	    if (homeFile == null) {  
	        // First fall-back. See if current directory is a bin directory  
	        // in a normal Tomcat install
	        File bootstrapJar = new File(userDir, "bootstrap.jar");  
	        if (bootstrapJar.exists()) {  
	            File f = new File(userDir, "..");  
	            try {  
	                homeFile = f.getCanonicalFile();  
	            } catch (IOException ioe) {  
	                homeFile = f.getAbsoluteFile();  
	            }  
	        }  
	    }  
  
	    if (homeFile == null) {  
	        // Second fall-back. Use current directory  
	        File f = new File(userDir);  
	        try {  
	            homeFile = f.getCanonicalFile();  
	        } catch (IOException ioe) {  
	            homeFile = f.getAbsoluteFile();  
	        }  
	    }  
  
	    catalinaHomeFile = homeFile;  
	    System.setProperty(  
	            Constants.CATALINA_HOME_PROP, catalinaHomeFile.getPath());  
  
	    // Then base  
	    String base = System.getProperty(Constants.CATALINA_BASE_PROP);  
	    if (base == null) {  
	        catalinaBaseFile = catalinaHomeFile;  
	    } else {  
	        File baseFile = new File(base);  
	        try {  
	            baseFile = baseFile.getCanonicalFile();  
	        } catch (IOException ioe) {  
	            baseFile = baseFile.getAbsoluteFile();  
	        }  
	        catalinaBaseFile = baseFile;  
	    }  
	    System.setProperty(  
	            Constants.CATALINA_BASE_PROP, catalinaBaseFile.getPath());  
	}
}
```
# Main()

```java
public static void main(String args[]) {  
      synchronized (daemonLock) {  // Object daemonLock = new Object();
        if (daemon == null) {  
            // Don't set daemon until init() has completed  
            Bootstrap bootstrap = new Bootstrap();  
            try {
                bootstrap.init();  
            } catch (Throwable t) {  
                handleThrowable(t);  
                t.printStackTrace();  
                return;            }  
            daemon = bootstrap;  
        } else {  
            // When running as a service the call to stop will be on a new  
            // thread so make sure the correct class loader is used to            // prevent a range of class not found exceptions.            Thread.currentThread().setContextClassLoader(daemon.catalinaLoader);  
        }  
    }  
    
    try {  
        String command = "start";  
        if (args.length > 0) {  
            command = args[args.length - 1];  
        }  
  
        if (command.equals("startd")) {  
            args[args.length - 1] = "start";  
            daemon.load(args);  
            daemon.start();  
        } else if (command.equals("stopd")) {  
            args[args.length - 1] = "stop";  
            daemon.stop();  
        } else if (command.equals("start")) {  
            daemon.setAwait(true);  
            daemon.load(args);  
            daemon.start();  
            if (null == daemon.getServer()) {  
                System.exit(1);  
            }  
        } else if (command.equals("stop")) {  
            daemon.stopServer(args);  
        } else if (command.equals("configtest")) {  
            daemon.load(args);  
            if (null == daemon.getServer()) {  
                System.exit(1);  
            }  
            System.exit(0);  
        } else {  
            log.warn("Bootstrap: command \"" + command + "\" does not exist.");  
        }  
    } catch (Throwable t) {  
        // Unwrap the Exception for clearer error reporting  
        if (t instanceof InvocationTargetException &&  
                t.getCause() != null) {  
            t = t.getCause();  
        }  
        handleThrowable(t);  
        t.printStackTrace();  
        System.exit(1);  
    }  
}
```

# 主要方法

## init() : void

```java
public void init() throws Exception {  
    initClassLoaders();  
  
    Thread.currentThread().setContextClassLoader(catalinaLoader);  
  
    SecurityClassLoad.securityClassLoad(catalinaLoader);  
  
    // Load our startup class and call its process() method  
    if (log.isDebugEnabled()) {  
        log.debug("Loading startup class");  
    }  
    Class<?> startupClass = catalinaLoader.loadClass("org.apache.catalina.startup.Catalina");  
    Object startupInstance = startupClass.getConstructor().newInstance();  
  
    // Set the shared extensions class loader  
    if (log.isDebugEnabled()) {  
        log.debug("Setting startup class properties");  
    }  
    String methodName = "setParentClassLoader";  
    Class<?> paramTypes[] = new Class[1];  
    paramTypes[0] = Class.forName("java.lang.ClassLoader");  
    Object paramValues[] = new Object[1];  
    paramValues[0] = sharedLoader;  
    Method method =  
        startupInstance.getClass().getMethod(methodName, paramTypes);  
    method.invoke(startupInstance, paramValues);  
  
    catalinaDaemon = startupInstance;  
}
```

**相关方法**

**1.  initClassLoaders() : void**

```java
private void initClassLoaders() {  
    try {  
        commonLoader = createClassLoader("common", null);  
        if (commonLoader == null) {  
            // no config file, default to this loader - we might be in a 'single' env.  
            commonLoader = this.getClass().getClassLoader();  
        }  
        catalinaLoader = createClassLoader("server", commonLoader);  
        sharedLoader = createClassLoader("shared", commonLoader);  
    } catch (Throwable t) {  
        handleThrowable(t);  
        log.error("Class loader creation threw exception", t);  
        System.exit(1);  
    }  
}
```

**相关方法**

**1.1  createClassLoader(String name, ClassLoader parent) : ClassLoader**

```java
private ClassLoader createClassLoader(String name, ClassLoader parent) throws Exception {  
    String value = CatalinaProperties.getProperty(name + ".loader");  
    if ((value == null) || (value.equals(""))) {  
        return parent;  
    }  
  
    value = replace(value);  
  
    List<Repository> repositories = new ArrayList<>();  
  
    String[] repositoryPaths = getPaths(value);  
  
    for (String repository : repositoryPaths) {  
        // Check for a JAR URL repository  
        try {  
            @SuppressWarnings("unused")  
            URL url = new URL(repository);  
            repositories.add(new Repository(repository, RepositoryType.URL));  
            continue;        } catch (MalformedURLException e) {  
            // Ignore  
        }  
  
        // Local repository  
        if (repository.endsWith("*.jar")) {  
            repository = repository.substring  
                (0, repository.length() - "*.jar".length());  
            repositories.add(new Repository(repository, RepositoryType.GLOB));  
        } else if (repository.endsWith(".jar")) {  
            repositories.add(new Repository(repository, RepositoryType.JAR));  
        } else {  
            repositories.add(new Repository(repository, RepositoryType.DIR));  
        }  
    }  
  
    return ClassLoaderFactory.createClassLoader(repositories, parent);  
}
```

