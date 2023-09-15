
设置工厂
```java
SAXParserFactory factory = SAXParserFactory.newInstance();
factory.setNamespaceAware(...);
factory.setValidating(...);
factory.setFeature(..., boolean)


```
生成并设置getXMLReader实例
```java
parser = getFactory().newSAXParser();

reader.setDTDHandler(this);  
reader.setContentHandler(this);
reader.setEntityResolver(entityResolver);
reader.setErrorHandler(this);
reader.setProperty("http://xml.org/sax/properties/lexical-handler", this);
```


# SAXParserFactory

## 属性
```java
setNamespaceAware
setFeature
setValidating
```



# 附录

## SAX和APACHE提供的所有feature

1）命名空间相关

| feature                                                                | 功能                                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `http://xml.org/sax/features/namespaces`                               | 打开、关闭名空间处理功能。                                                                                                                                                                                                                                                                 |
| `http://xml.org/sax/features/namespace-prefixes`                       | 报告、不报告名空间前缀。                                                                                                                                                                                                                                                                   |
| `http://xml.org/sax/features/string-interning`                         | 是否将所有的名字等字符串内部化，即使用String.intern()方法处理所有的名字字符串，Xerces目前不支持这个特性，在支持这种特性的解析器上这样可以节省内存空间，但是可能会稍微降低速度。在处理有很多的重复tag的时候打开这个特性可以节约很多空间；由于节省了重新分配内存的时间，反而可能会提高速度。 |
| `http://xml.org/sax/features/external-general-entities`                | 是否包含外部生成的实体。                                                                                                                                                                                                                                                                   |
| `http://xml.org/sax/features/external-parameter-entities`              | 是否包含外部的参数，包括外部DTD子集。                                                                                                                                                                                                                                                      |
| `http://apache.org/xml/features/allow-java-encodings`                  | 是否允许在XMLDecl和TextDecl使用java的字符编码名。如果设置为false则在遇到java字符编码名的时候会产生一个错误。需要注意的是不是所有的解析器都会允许使用java字符编码名的。                                                                                                                     |
| `http://apache.org/xml/features/continue-after-fatal-error`            | 是否在发生致命错误后继续进行解析。                                                                                                                                                                                                                                                         |
| `http://apache.org/xml/features/nonvalidating/load-dtd-grammar`        | 是否装载DTD语法并且自动增添DTD中定义的缺省值。若`http://xml.org/sax/features/validation`设置为true则此特性自动设置为true。                                                                                                                                                                 |
| `http://apache.org/xml/features/dom/defer-node-expansion`              | 这个特性是DOM特性，在这里一起介绍了。是否使用懒惰型节点展开，当这个特性设置为true时，可以提高解析速度并节约内存。这个特性同属性`http://apache.org/xml/properties/dom/document-class-name`的设置有关。                                                                                      |
| `http://apache.org/xml/features/dom/create-entity-ref-nodes`           | 这个特性是DOM特性，是否用引用的方式建立实体节点，若设置为true则会建立EntityReference节点，若设置为false则会用实际字符串取代实体引用。                                                                                                                                                      |
| `http://apache.org/xml/features/dom/include-ignorable-whitespace`      | 这个特性是DOM特性，是否将可以忽略的空白字符串包含在DOM树里面，缺省为true。但是笔者本人一般情况下会设置为false。另外仅仅在打开了校验的情况下才可以判断出来是否有空白字符串。因此这个特性是同`http://xml.org/sax/features/validation`相关的。                                                |

2）验证相关

| feature                                                                | 功能                                                                                                                                                    |
| ---------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `http://xml.org/sax/features/validation`                               | 是否打开校验。当关闭校验的时候可以大大节约内存空间并且大大提高解析速度。因此如果使用的XML文档是可靠的，例如程序生成的，最好关闭校验。                   |
| `http://apache.org/xml/features/validation/schema`                     | 是否使用schema。这个特性是apache为Xerces提供的。                                                                                                        |
| `http://apache.org/xml/features/validation/dynamic`                    | 当设置为true时，仅仅在XML文档指明语法时进行校验，若设置为false，则由`http://xml.org/sax/features/validation`决定，若其为false则不校验，若为true则校验。 |
| `http://apache.org/xml/features/validation/warn-on-duplicate-attdef`   | 是否在遇到重复的属性声明时警告。                                                                                                                        |
| `http://apache.org/xml/features/validation/warn-on-undeclared-elemdef` | 否在遇到未定义的元素的时候警告。                                                                                                                        |

3）杂项


# ContentHandler

## 方法

```java
public void startDocument () throws SAXException;
public void endDocument() throws SAXException;

public void startElement(String uri, String localName, String qName, Attributes atts) throws SAXException;
```