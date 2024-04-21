## 前言

<font color=FFFF00>黄色</font>字体指自己理解部分。

<font color=FFFF00>通过schema的方式来定义和使用xml，首先需要在根元素属性中声明命名空间</font>。



## Schema的定义

一个 schema 的定义需要使用`schema`根元素。

### 1、XSD头部信息

`<schema>` 元素是每一个 XML Schema 的根元素，需要在根元素中定义命名空间。

```xml schema
<?xml version="1.0"?>
<xsd:schema>

...
...

</xsd:schema>
```

一个 schema 声明往往长下面这样。

```xml
<?xml version="1.0"?>
 <xsd:schema 
 xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
 xmlns="http://www.springframework.org/schema/context" 
 targetNamespace="http://www.springframework.org/schema/context" 
 elementFormDefault="qualified" 
 attributeFormDefault="unqualified">
...
...
</xsd:schema>
```

**代码解释**

1）`xmlns:xsd="http://www.w3.org/2001/XMLSchema"` 

- 显示 schema 中用到的元素和数据类型来自命名空间 `"http://www.w3.org/2001/XMLSchema"`。
- 规定了来自命名空间 ` "http://www.w3.org/2001/XMLSchema" `的元素和数据类型应该使用前缀 `xs`。
- 这个属性是通用的属性和属性值。

2）`xmlns="http://www.springframework.org/schema/context" `

- 指出默认的命名空间是`"http://www.springframework.org/schema/context" `。

3）`targetNamespace="http://www.springframework.org/schema/context" `

- 显示被此schema定义的元素来自命名空间` "http://www.springframework.org/schema/context"`。

4）`elementFormDefault="qualified"`  &&  `attributeFormDefault="unqualified"`

- elementFormDefault和attributeFormDefault有两个属性值：qualified、unqualified
- elementFormDefault=‘unqualified’ 时表示子元素不必须使用命名空间前缀，但这不等于说这些子元素是属于无命名空间，而是从属于父顶级元素的目标命名空间。

- elementFormDefault=‘qualified’ 时表示子元素必须使用命名空间前缀, 当然, 这些子元素是位于目标命名空间之下。
- attributeFormDefault=‘unqualified’ 时表示目标命名空间下的这个属性不要带命名空间前缀。
- attributeFormDefault=‘qualified’ 时表示来自目标命名空间下的属性必须要用命名空间前缀修饰。




## 自定义Schema的使用

```xml
<?xml version="1.0" encoding="UTF-8"?>
<property-placeholder 
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="
		http://www.springframework.org/schema/context
		http://www.springframework.org/schema/context/spring-context.xsd"

>

...

</property-placeholder>
```

**代码解释**

1）`xmlns:context="http://www.springframework.org/schema/context"`

- 规定使用的命名空间，并且使用空间中的元素前缀用context。

2）`xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"`

- 规定使用的命名空间，并且使用空间中的元素前缀用xsi。

3）`xsi:schemaLocation="http://www.springframework.org/schema/context
		                    http://www.springframework.org/schema/context/spring-context.xsd"`

- 使用xsi命名空间中的属性schemaLocation，其中的第一个属性是需要使用的命名空间，第二个属性是提供命名空间使用的xml schema的位置。



## XSD中定义的内容

1. 定义可出现在文档中的元素
2. 定义可出现在文档中的属性
3. 定义哪个元素是子元素
4. 定义子元素的次序
5. 定义子元素的数目
6. 定义元素是否为空，或者是否可包含文本
7. 定义元素和属性的数据类型
8. 定义元素和属性的默认值以及固定值



### 1、元素的定义

<xsd:element name="xxx" type="yyy"/>



### 2、元素属性的定义

<xsd:attribute name="xxx" type="yyy"/>

元素的限定
格式：

<xsd:element name="age">
	<xsd:simpleType>
	  <xsd:restriction base="xs:integer">
	    <xsd:minInclusive value="0"/>
	    <xsd:maxInclusive value="120"/>
	  </xsd:restriction>
	</xsd:simpleType>
</xs:element> 
1
2
3
4
5
6
7
8
数据类型的限定

限定	描述
enumeration	定义可接受值的一个列表
fractionDigits	定义所允许的最大的小数位数。必须大于等于0。
length	定义所允许的字符或者列表项目的精确数目。必须大于或等于0。
maxExclusive	定义数值的上限。所允许的值必须小于此值。
maxInclusive	定义数值的上限。所允许的值必须小于或等于此值。
maxLength	定义所允许的字符或者列表项目的最大数目。必须大于或等于0。
minExclusive	定义数值的下限。所允许的值必需大于此值。
minInclusive	定义数值的下限。所允许的值必需大于或等于此值。
minLength	定义所允许的字符或者列表项目的最小数目。必须大于或等于0。
pattern	定义可接受的字符的精确序列。
totalDigits	定义所允许的阿拉伯数字的精确位数。必须大于0。
whiteSpace	定义空白字符（换行、回车、空格以及制表符）的处理方式。
XSD中各个元素和属性以及含义
XSD 元素

元素	解释
all	规定子元素能够以任意顺序出现，每个子元素可出现零次或一次。
annotation	annotation 元素是一个顶层元素，规定 schema 的注释。
any	使创作者可以通过未被 schema 规定的元素来扩展 XML 文档。
anyAttribute	使创作者可以通过未被 schema 规定的属性来扩展 XML 文档。
appInfo	规定 annotation 元素中应用程序要使用的信息。
attribute	定义一个属性。
attributeGroup	定义在复杂类型定义中使用的属性组。
choice	仅允许在 声明中包含一个元素出现在包含元素中。
complexContent	定义对复杂类型（包含混合内容或仅包含元素）的扩展或限制。
complexType	定义复杂类型。
documentation	定义 schema 中的文本注释。
element	定义元素。
extension	扩展已有的 simpleType 或 complexType 元素。
field	规定 XPath 表达式，该表达式规定用于定义标识约束的值。
group	定义在复杂类型定义中使用的元素组。
import	向一个文档添加带有不同目标命名空间的多个 schema。
include	向一个文档添加带有相同目标命名空间的多个 schema。
key	指定属性或元素值（或一组值）必须是指定范围内的键。
keyref	规定属性或元素值（或一组值）对应指定的 key 或 unique 元素的值。
list	把简单类型定义为指定数据类型的值的一个列表。
notation	描述 XML 文档中非 XML 数据的格式。
redefine	重新定义从外部架构文件中获取的简单和复杂类型、组和属性组。
restriction	定义对 simpleType、simpleContent 或 complexContent 的约束。
schema	定义 schema 的根元素。
selector	指定 XPath 表达式，该表达式为标识约束选择一组元素。
sequence	要求子元素必须按顺序出现。每个子元素可出现 0 到任意次数。
simpleContent	包含对 complexType 元素的扩展或限制且不包含任何元素。
simpleType	定义一个简单类型，规定约束以及关于属性或仅含文本的元素的值的信息。
union	定义多个 simpleType 定义的集合。
unique	指定属性或元素值（或者属性或元素值的组合）在指定范围内必须是唯一的。
element ：定义元素
元素信息：

出现次数	在架构中定义的元素的数目。
父元素	schema、choice、all、sequence
内容	simpleType、complexType、key、keyref、unique
属性值：

属性	属性值	含义
name	自定义	设置元素的名字
default	自定义	设置默认值
fixed	自定义	设置固定值，设置后不允许修改
id	自定义	规定该元素唯一的id
ref	自定义	对另一个元素的引用
substitutionGroup	自定义	规定可用来替代该元素的元素的名称。 该元素必须具有相同的类型或从指定元素类型派生的类型。
maxOccurs	自定义	规定 element 元素在父元素中可出现的最大次数。
minOccurs	自定义	规定 element 元素在父元素中可出现的最小次数。
abstract	false	指示元素是否可以在实例文档中使用。如果该值为 true，则元素不能出现在实例文档中。 相反，substitutionGroup 属性包含该元素的限定名 (QName) 的其他元素必须出现在该元素的位置。多个元素可以在其 substitutionGroup 属性中引用该元素。默认值是 false。
block	extension	防止通过扩展派生的元素被用来替代该元素。
restriction	防止通过限制派生的元素被用来替代该元素。
substitution	防止通过替换派生的元素被用来替代该元素。
#all	防止所有派生的元素被用来替代该元素。
final	extension	防止通过扩展派生的元素被用来替代该元素
restriction	防止通过限制派生的元素被用来替代该元素
#all	防止所有派生的元素被用来替代该元素
form	unqualified	则无须通过命名空间前缀限定该元素。
qualified	必须通过命名空间前缀限定该元素。
type	string	规定元素的值为字符串类型
decimal	规定元素的值为小数类型
integer	规定元素的值为整数类型
boolean	规定元素的值为布尔型
date	规定元素的值为日期类型，实现方式：YYYY-MM-DD
time	规定元素的值为时间类型，实现方式：HH:MM:SS
attribute ：定义元素的属性
元素信息

出现次数	在 schema 元素中定义一次， 在复杂类型或属性组中引用多次
父元素	attributeGroup、schema、complexType、restriction (simpleContent)、extension (simpleContent)、restriction (complexContent)、extension (complexContent)
内容	annotation、simpleType
属性	属性值	含义
id	自定义	规定该元素唯一的id
name	自定义	设置属性的名字
default	自定义	设置默认值
fixed	自定义	设置固定值，设置后不允许修改
form	unqualified	指示此属性无须由命名空间前缀限定，且无须匹配此属性的无冒号名称 (NCName)，即本地名称。
qualified	指示必须通过命名空间前缀和该属性的无冒号名称 (NCName) 来限定此属性。
use	optional	属性是可选的并且可以具有任何值（默认）
prohibited	不能使用属性。
required	属性的必需的。
type	string	规定属性的值为字符串类型
decimal	规定属性的值为小数类型
integer	规定属性的值为整数类型
boolean	规定属性的值为布尔型
date	规定属性的值为日期类型，实现方式：YYYY-MM-DD
time	规定属性的值为时间类型，实现方式：HH:MM:SS
$$元素type属性值集合链接



## 附录

### ElementFormDefault & AttribuateFormDefault

1、在xml中，所有引用xsd的全局的元素都必须加上命名空间的前缀

（例如xmlns:aa=http://www.example.org/classroom，全局元素都得加上aa）。

2、非全局的元素当设置为qualified时，必须添加命名空间的前缀。

3、非全局的元素当设置为unqualified时，不必也不能添加前缀。

 

下面是一个简单的例子：

a.  当设置为unqualified时，user为全局元素（可作为根元素）必须添加前缀，非全局元素

（id，name）不必添加前缀。







b.  当设置为qualified时，所有的元素都必须添加前缀。




其实elementFormDefault的qualified/unqualified还与schema的设计模式有关系，目前常用的有Russian Roll , Salami Slice , Venetian Blind

详细及最佳实践可以参照：http://www.xfront.com/GlobalVersusLocal.html#BestPractice

 

Salami Slice这种schema的设计模式，将所有的元素设置为全局元素，设置不设置elementFormDefault是没有任何意义的，文章的开通说过，所有的全局元素必须添加前缀。

http://www.xfront.com/GlobalVersusLocal.html#BestPractice

上网站中提到的Venetian Blind的优势之一：

UseelementFormDefault to act as a switch for controlling namespaceexposure - if you want element namespaces exposed in instance documents, simplyturn the elementFormDefault switch to "on" (i.e, setelementFormDefault= "qualified"); if you don't want elementnamespaces exposed in instance documents, simply turn the elementFormDefaultswitch to "off" (i.e., setelementFormDefault="unqualified").
