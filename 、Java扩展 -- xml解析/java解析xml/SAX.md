## 1. SAX简介

SAX解析器在解析XML输入数据的各个组成部分时会报告事件，但不会以任何方式存储文档，而是由事件处理器建立相应的数据结构。实际上DOM解析器是在SAX解析器的基础上构建的，它在接收到解析器事件时构建DOM树。

## 2. 使用SAX解析器

![img](https:////upload-images.jianshu.io/upload_images/5336514-3ceb8d56c3576362.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/401/format/webp)

SAX解析器架构图

在使用SAX解析器时，需要一个处理器来为各种解析器事件定义事件动作。DefaultHandler接口定义了若干个在解析文档时解析器会调用的回调方法。下面是最重要的几个方法：

- startElement和endElement在每当遇到起始或终止标签时调用。
- characters在每当遇到字符数据时调用。
- startDocument和endDocument分别在文档开始和结束时各调用一次。

例如，在解析以下片段时：

```xml
<person>
    <name type="string">韩信</name>
    <age>25</age>
</person>
```

解析器会产生以下回调：

- startElement，元素名：person
- startElement，元素名：name ，属性：type="string"
- characters，内容：韩信
- endElement，元素名：name
- startElement，元素名：age
- characters，内容：25
- endElement，元素名：age
- endElement，元素名：person

处理器必须覆盖这些方法，让它们执行在解析文件时我们想让它们执行的动作。



下面通过一个简单的demo体会SAX解析XML的过程。

### 2.1 准备xml

首先准备person.xml，内容如下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persons>
    <person>
      <name>韩信</name>
      <age>25</age>
   </person>
   <person>
      <name>李白</name>
      <age>23</age>
   </person>
</persons>
```

### 2.2 解析代码

- 获取解析工厂
- 从解析工厂获取解析器
- 得到解读器
- 设置内容处理器
- 读取xml的文档内容



```java
package cn.lastwhisper.javabasic.Sax;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;
import org.xml.sax.helpers.DefaultHandler;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.IOException;

/**
 * @desc SAX方式解析xml文件
 */
public class SaxTest {
    public static void main(String[] args) throws ParserConfigurationException, SAXException, IOException {
        // 1.获取解析工厂
        SAXParserFactory factory = SAXParserFactory.newInstance();
        // 2.从解析工厂获取解析器
        SAXParser parser = factory.newSAXParser();
        // 3.得到解读器
        XMLReader reader=parser.getXMLReader();
        // 4.设置内容处理器
        reader.setContentHandler(new PHandler());
        // 5.读取xml的文档内容
        reader.parse("src/main/java/cn/lastwhisper/javabasic/Sax/person.xml");

    }

}

class PHandler extends DefaultHandler {
    /**
     * @desc 文档解析开始时调用，该方法只会调用一次
     * @param
     * @return void
     */
    @Override
    public void startDocument() throws SAXException {
        System.out.println("----解析文档开始----");
    }

    /**
     * @desc 每当遇到起始标签时调用
     * @param uri xml文档的命名空间
     * @param localName 标签的名字
     * @param qName 带命名空间的标签的名字
     * @param attributes 标签的属性集
     * @return void
     */
    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        System.out.println("标签<"+qName + ">解析开始");
    }

    /**
     * @desc 解析标签内的内容的时候调用
     * @param ch 当前读取到的TextNode(文本节点)的字节数组
     * @param start 字节开始的位置，为0则读取全部
     * @param length 当前TextNode的长度
     * @return void
     */
    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        String contents = new String(ch, start, length).trim();
        if (contents.length() > 0) {
            System.out.println("内容为-->" + contents);
        } else {
            System.out.println("内容为-->" + "空");
        }
    }
    /**
     * @desc 每当遇到结束标签时调用
     * @param uri xml文档的命名空间
     * @param localName 标签的名字
     * @param qName 带命名空间的标签的名字
     * @return void
     */
    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        System.out.println("标签</"+qName + ">解析结束");
    }
    /**
     * @author lastwhisper
     * @desc 文档解析结束后调用，该方法只会调用一次
     * @param
     * @return void
     */
    @Override
    public void endDocument() throws SAXException {
        System.out.println("----解析文档结束----");
    }
}
```

运行main函数，查看运行结果

![img]()

运行结果

## 3.3 分析解析过程

----解析文档开始---- 调用startDocument

标签<persons>解析开始 调用startElement

内容为-->空     调用characters，因为<persons></persons>之间只有标签没有内容，所以为空。

标签<person>解析开始 调用startElement 

内容为-->空     调用characters，因为<person></person>之间只有标签没有内容，所以为空。 

标签<name>解析开始    调用startElement 

内容为-->韩信    调用characters，<name>韩信</name>之间内容为：韩信 标签</name>

解析结束   调用endElement 



内容为-->空     调用characters，每次执行完调用endElement之后会调用一次characters 标签<age>解析开始 调用startElement 内容为-->25        调用characters，<age>25</age>之间内容为：25 标签</age>解析结束 调用endElement 内容为-->空     调用characters，每次执行完调用endElement之后会调用一次characters 标签</person>解析结束 调用endElement 内容为-->空     调用characters，每次执行完调用endElement之后会调用一次characters ... 标签<person>解析开始 内容为-->空 标签<name>解析开始 内容为-->李白 标签</name>解析结束 内容为-->空 标签<age>解析开始 内容为-->23 标签</age>解析结束 内容为-->空 标签</person>解析结束 内容为-->空 标签</persons>解析结束 ----解析文档结束---- 调用endDocument，xml解析结束

# 4. 解析xml到pojo对象

待解析的xml，person.xml



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persons>
    <person>
      <name>韩信</name>
      <age>25</age>
   </person>
   <person>
      <name>李白</name>
      <age>23</age>
   </person>
</persons>
```

xml转换为pojo的代码



```java
package cn.lastwhisper.javabasic.Sax;

import org.xml.sax.Attributes;
import org.xml.sax.SAXException;
import org.xml.sax.XMLReader;
import org.xml.sax.helpers.DefaultHandler;

import javax.xml.parsers.ParserConfigurationException;
import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

/**
 * @author lastwhisper
 * @desc 将xml数据解析到pojo中
 *
 * @email gaojun56@163.com
 */
public class SaxXmlToPojo {
    public static void main(String[] args) throws ParserConfigurationException, SAXException, IOException {
        // SAX解析
        // 1.获取解析工厂
        SAXParserFactory factory = SAXParserFactory.newInstance();
        // 2.从解析工厂获取解析器
        SAXParser parse = factory.newSAXParser();
        // 3.得到解读器
        XMLReader reader = parse.getXMLReader();
        // 4.设置内容处理器
        PersonHandler personHandler = new PersonHandler();
        reader.setContentHandler(personHandler);
        // 5.读取xml的文档内容
        reader.parse("src/main/java/cn/lastwhisper/javabasic/Sax/person.xml");

        List<Person> persons = personHandler.getPersons();
        for (Person person : persons) {
            System.out.println("姓名：" + person.getName() + " 年龄：" + person.getAge());
        }
    }

}

class PersonHandler extends DefaultHandler {
    private List<Person> persons;
    private Person person;
    private String tag; // 存储操作标签

    /**
     * @author lastwhisper
     * @desc 文档解析开始时调用，该方法只会调用一次
     * @param
     * @return void
     */
    @Override
    public void startDocument() throws SAXException {
        persons = new ArrayList<Person>();
    }

    /**
     * @author lastwhisper
     * @desc 标签（节点）解析开始时调用
     * @param uri xml文档的命名空间
     * @param localName 标签的名字
     * @param qName 带命名空间的标签的名字
     * @param attributes 标签的属性集
     * @return void
     */
    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        tag = qName;
        if ("person".equals(tag)) {
            person = new Person();
        }
    }

    /**
     * @author lastwhisper
     * @desc 解析标签的内容的时候调用
     * @param ch  字符
     * @param start 字符数组中的起始位置
     * @param length 要从字符数组使用的字符数
     * @return void
     */
    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        String contents = new String(ch, start, length).trim();
        if ("name".equals(tag)) {
            person.setName(contents);
        } else if ("age".equals(tag)) {
            if (contents.length() > 0) {
                person.setAge(Integer.valueOf(contents));
            }
        }
    }

    /**
     * @author lastwhisper
     * @desc 标签（节点）解析结束后调用
     * @param uri xml文档的命名空间
     * @param localName 标签的名字
     * @param qName 带命名空间的标签的名字
     * @return void
     */
    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        if ("person".equals(qName)) {
            persons.add(person);
        }
        tag = null; //tag丢弃了
    }

    /**
     * @author lastwhisper
     * @desc 文档解析结束后调用，该方法只会调用一次
     * @param
     * @return void
     */
    @Override
    public void endDocument() throws SAXException {
    }

    public List<Person> getPersons() {
        return persons;
    }
}
```



作者：最后的轻语_dd43
链接：https://www.jianshu.com/p/8df626ea70ed
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。