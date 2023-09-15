
CDS全称Class-Data Sharing


# 使用JDK归档文件

# 使用应用类归档文件

为了实际共享应用程序的类，

## 生成list文件

生成方式

byClass.lst文件中没有org.example.CDSMain项。
byJar.lst文件中有org.example.CDSMain项。

| 使用的lst文件 | 运行方式 | --class-path      | 名称     | 结果 |
| ------------- |:--------:| ----------------- | -------- | ---- |
| byClass.lst   |    -     | java_internal.jar | jsa1.jsa | 正常 |
| byClass.lst   |    -     | -                 | jsa2.jsa | 正常 |
| byJar.lst     |    -     | java_internal.jar | jsa3.jsa | 正常 |
| byJar.lst     |    -     | -                 | jsa4.jsa | 警告 |
|               |          |                   |          |      |


```bash
## 1
# java -Xshare:dump -XX:SharedClassListFile="D:\Tmp\byClass.lst" -XX:SharedArchiveFile="D:\Tmp\jsa1.jsa" --class-path "D:\Workspace\IDEA\java_se\java_internal\java_internal.jar
## 2
# java -Xshare:dump -XX:SharedClassListFile="D:\Tmp\byClass.lst" -XX:SharedArchiveFile="D:\Tmp\jsa2.jsa"
## 3
# java -Xshare:dump -XX:SharedClassListFile="D:\Tmp\byJar.lst" -XX:SharedArchiveFile="D:\Tmp\jsa3.jsa" --class-path "D:\Workspace\IDEA\java_se\java_internal\java_internal.jar"
## 4
# D:\Workspace\IDEA\java_se\java_internal\module1\src\java\org\example>java -Xshare:dump -XX:SharedClassListFile="D:\Tmp\byJar.lst" -XX:SharedArchiveFile="D:\Tmp\jsa4.jsa"
# 报错[0.118s][warning][cds] Preload Warning: Cannot find org/example/CDSMain
## 4

```

# 结果

| 使用的jsa文件 | 运行方式 | 结果     |
| ------------- | -------- | -------- |
| jsa1.jsa      | class    | 来自文件 |
| jsa1.jsa      | jar      | 来自文件 |
| jsa2.jsa      | class    | 来自文件 |
| jsa2.jsa      | jar      | 来自文件 |
| jsa3.jsa      | class    | 来自文件 |
| jsa3.jsa      | jar      | 来自共享 |
|               |          |          |

```bash
# 1
java -Xlog:class+load:file="D:\Tmp\log.log" -XX:SharedArchiveFile="D:\Tmp\jsa1.jsa" CDSMain.java
# 2
java -Xlog:class+load:file="D:\Tmp\log.log" -XX:SharedArchiveFile="D:\Tmp\jsa1.jsa" -jar "D:\Workspace\IDEA\java_se\java_internal\java_internal.jar"
# 3
java -Xlog:class+load:file="D:\Tmp\log.log" -XX:SharedArchiveFile="D:\Tmp\jsa2.jsa" CDSMain.java
# 4
java -Xlog:class+load:file="D:\Tmp\log.log" -XX:SharedArchiveFile="D:\Tmp\jsa2.jsa" -jar "D:\Workspace\IDEA\java_se\java_internal\java_internal.jar"
# 5
java -Xlog:class+load:file="D:\Tmp\log.log" -XX:SharedArchiveFile="D:\Tmp\jsa3.jsa" CDSMain.java
```