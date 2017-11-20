# XML:（EXtensible Markup Language）可扩展编辑语言
请参考 [XML](http://www.w3school.com.cn/xml/xml_intro.asp)
## 1.简介
- XML是一种标记语言，很类似HTML；
- XML的设计宗旨是**传输数据**，而非显示数据；
>- XML 与 HTML 的主要差异
>- XML 不是 HTML 的替代。
>- XML 和 HTML 为不同的目的而设计：
>- XML 被设计为传输和存储数据，其焦点是数据的内容。
>- HTML 被设计用来显示数据，其焦点是数据的外观。
>- HTML 旨在显示信息，而 XML 旨在传输信息。
>- XML 应用于 web 开发的许多方面，常用于简化数据的存储和共享。
>- XML以文本的格式存储，提供了一种独立与软件和硬件的数据存储方式；

## 2.结构
XML 文档形成了一种树结构，它从“根部”开始，然后扩展到“枝叶”。
下面介绍一个XML文档事例：
```XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<note>
<to>George</to>
<from>John</from>
<heading>Reminder</heading>
<body>Don't forget the meeting!</body>
</note>
```
- 第一行是XML声明，定义了XML版本，和使用的编码；
- 下一行描述文档的艮元素`<note>`;
- 接下来 4 行描述根的 4 个子元素`<to> <from> <body>`;
- 最后一行定义根元素的结尾：`</note>`;  
 ![实例](http://www.w3school.com.cn/i/ct_nodetree1.gif)
## 3.语法
- XML元素必须有关闭标签`<p>This is a paragraph</p>`
- XML标签对大小写敏感
- XML 必须正确地嵌套
- XML 文档必须有根元素
- XML 的属性值须加引号,如`date`的值
	```XML
	<note date="08/08/2008">
	<to>George</to>
	<from>John</from>
	</note> 
	```
- 实体引用：在 XML 中，一些字符拥有特殊的意义;

	| `XML`  | 符号  |
	| :------| -----:|
	| `&gt`  | `>`  |
	| `&lt`| `<`|
	| `&amp ` |`&` |
	| `&apos` |`'` |
	| `&quot`| `"`|
- XML 中的注释`< !-- This is a comment -- > `
- XML中空格会被保留
- XML以LF存储换行
## 4.元素
XML 元素指的是从（且包括）开始标签直到（且包括）结束标签的部分。 元素可包含其他元素、文本或者两者的混合物。元素也可以拥有属性。
```XML
<bookstore>
<book category="CHILDREN">
<title>Harry Potter</title> 
<author>J K. Rowling</author> 
<year>2005</year> 
<price>29.99</price> 
</book>
<book category="WEB">
<title>Learning XML</title> 
<author>Erik T. Ray</author> 
<year>2003</year> 
<price>39.95</price> 
</book>
</bookstore>
```
- 在上例中，`<bookstore>` 和 `<book>` 都拥有元素内容，因为它们包含了其他元素。`<author> `只有文本内容，因为它仅包含文本。
- 在上例中，只有 `<book>` 元素拥有属性 `category="CHILDREN"`。
- XML 命名规则
	- 名称可以含字母、数字以及其他的字符
	- 名称不能以数字或者标点符号开始
	- 名称不能以字符 “xml”（或者 XML、- Xml）开始
	- 名称不能包含空格
## 5.XML属性
XML 元素可以在开始标签中包含属性，类似 HTML。属性 (Attribute) 提供关于元素的额外（附加）信息。
- 属性值必须被引号包围，不过单引号和双引号均可使用，如果属性值本身包含双引号，那么有必要使用单引号包围它；
- XML 中，应该尽量避免使用属性。如果信息感觉起来很像数据，那么请使用子元素；
- 有时候会向元素分配 ID 引用。这些 ID 索引可用于标识 XML 元素；
- 属性可以有多个，例如
	```XML
	<ctl name="Voice Rx Device Mute" id="0" value="0" />
	```