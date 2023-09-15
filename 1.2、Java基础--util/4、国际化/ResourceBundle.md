Java（J2SE）对国际化的基本实现——ResourceBundle，因为MessageSource是用它实现的。

# 基本使用

要使用ResourceBundle，必须按照规范的格式放置*.properties资源文件，然后根据输入的语言环境来返回资源。

按照开发文档的要求，使用ResourceBundle加载的资源文件都必须放置在根目录，并且必须按照`${name}_${language}_${region}`的方式来命名。这个命名方式正好能对应ResourceBundle::getBundle方法中的参数，例如 `ResourceBundle.getBundle("i18n", new Locale("zh", "CN"))` 。"i18n"对应`${name}`，"zh"定位`${language}`，而“CN”对应`${region}`。这样我们就可以通过传导参数来使用不同的资源。如果不指定`${language}`和`${region}`，该文件就是一个默认文件。

举个栗子：

我们有3个资源文件放置在 *classpath的根目录* 中，文件名分别为 `i18n_en_US.properties`、`i18n_zh_CN.properties`和 `i18n_web_BASE64.properties` 。文件中的内容如下：

```properties
#i18n_en_US.properties
say=Hallo world!

#i18n_zh_CN.properties
say=\u5927\u5BB6\u597D\uFF01

#i18n_web_BASE64.properties
say=+-+-+-ABC
```

然后通过ResourceBundle类来使用这些i18n的资源文件：

```java
//使用当前操作系统的语言环境
ResourceBundle rb = ResourceBundle.getBundle("i18n", Locale.getDefault());
System.out.println(rb.getString("say"));
//指定简体中文环境
rb = ResourceBundle.getBundle("i18n", new Locale("zh", "CN"));
System.out.println(rb.getString("say"));
//通过预设指定简体英文环境
rb = ResourceBundle.getBundle("i18n", Locale.SIMPLIFIED_CHINESE);
System.out.println(rb.getString("say"));
//指定美国英语
rb = ResourceBundle.getBundle("i18n", Locale.US);
System.out.println(rb.getString("say"));
//使用自定义的语言环境
Locale locale = new Locale("web", "BASE64");
rb = ResourceBundle.getBundle("i18n", locale);
System.out.println(rb.getString("say"));
```


# Locale类的预设类型

Locale类预设了很多资源类型，比如Locale.SIMPLIFIED_CHINESE、Locale.US，实际上他们就等价于_new Locale("zh", "CN")和new Locale_("en", "US")。只是Java的开发人员做了一些静态的预设。

除了预设内容的Locale，我们还可以像_Locale locale = new Locale("web", "BASE64")这样添加自定义的内容，他对应名为i18n_web_BASE64.properties的资源文件。
