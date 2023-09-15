
# Rule接口

# Rule实现类

## ObjectCreateRule
```java
// ("Server", "org.apache.catalina.core.StandardServer", "className")
// ("Server/GlobalNamingResources", "org.apache.catalina.deploy.NamingResourcesImpl")
// ("Server/Service", "org.apache.catalina.core.StandardService", "className")
// ("Server/Service/Listener", null, "className")
// ("Server/Service/Executor", "org.apache.catalina.core.StandardThreadExecutor", "className")
// ("Server/Service/Connector/SSLHostConfig", "org.apache.tomcat.util.net.SSLHostConfig")
// ("Server/Service/Connector/SSLHostConfig/OpenSSLConf", "org.apache.tomcat.util.net.openssl.OpenSSLConf")
// ("Server/Service/Connector/SSLHostConfig/OpenSSLConf/OpenSSLConfCmd", "org.apache.tomcat.util.net.openssl.OpenSSLConfCmd")
// ("Server/Service/Connector/Listener", null, "className");
// ("Server/Service/Connector/UpgradeProtocol", null, "className")
```




## SetPropertiesRule

```java
// ("Server")
// ("Server/GlobalNamingResources")
// ("Server/Listener")
// ("Server/Service")
// ("Server/Service/Listener")
// ----------------------- Executor
// ("Server/Service/Executor")
// ("Server/Service/Connector",new String[]{"executor", "sslImplementationName", "protocol"})
// ("Server/Service/Connector/SSLHostConfig")
// ("Server/Service/Connector/SSLHostConfig/Certificate", new String[]{"type"})
// ("Server/Service/Connector/SSLHostConfig/OpenSSLConf")
// ("Server/Service/Connector/SSLHostConfig/OpenSSLConf/OpenSSLConfCmd")
// ("Server/Service/Connector/Listener")
// ("Server/Service/Connector/UpgradeProtocol")


```

## SetNextRule

```java
("Server", "setServer", "org.apache.catalina.Server")
("Server/GlobalNamingResources", "setGlobalNamingResources", "org.apache.catalina.deploy.NamingResourcesImpl")
("Server/Listener", "addLifecycleListener", "org.apache.catalina.LifecycleListener")
("Server/Service", "addService", "org.apache.catalina.Service")
("Server/Service/Listener", "addLifecycleListener", "org.apache.catalina.LifecycleListener")
("Server/Service/Executor", "addExecutor", "org.apache.catalina.Executor")
("Server/Service/Connector", "addConnector", "org.apache.catalina.connector.Connector")
("Server/Service/Connector/SSLHostConfig", "addSslHostConfig", "org.apache.tomcat.util.net.SSLHostConfig")
("Server/Service/Connector/SSLHostConfig/Certificate", "addCertificate", "org.apache.tomcat.util.net.SSLHostConfigCertificate")
("Server/Service/Connector/SSLHostConfig/OpenSSLConf", "setOpenSslConf", "org.apache.tomcat.util.net.openssl.OpenSSLConf")
("Server/Service/Connector/SSLHostConfig/OpenSSLConf/OpenSSLConfCmd", "addCmd", "org.apache.tomcat.util.net.openssl.OpenSSLConfCmd")
("Server/Service/Connector/Listener", "addLifecycleListener", "org.apache.catalina.LifecycleListener")
("Server/Service/Connector/UpgradeProtocol", "addUpgradeProtocol", "org.apache.coyote.UpgradeProtocol")
```

## ConnectorCreateRule
```java
// "Server/Service/Connector"



```

## AddPortOffsetRule

```java
// "Server/Service/Connector"
```

## CertificateCreateRule
```java
"Server/Service/Connector/SSLHostConfig/Certificate"
```

## SetParentClassLoaderRule
```java
Server/Service/Engine
```