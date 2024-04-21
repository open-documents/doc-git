
在Java 9（jdk-9 + 170）默认情况下不允许应用程序查看来自JDK的所有类，而不像以前的所有Java版本。需要使用--add-exports和--add-opens配置参数。

```bash
--add-opens java.base/jdk.internal.loader=ALL-UNNAMED 
--add-opens jdk.zipfs/jdk.nio.zipfs=ALL-UNNAMED
--add-opens java.base/jdk.internal.misc=ALL-UNNAMED
```