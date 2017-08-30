## xml的有哪些解析技术？区别是什么？

| 解析方式 | 思想 | 优点 | 缺点 |
| :--- | :--- | :--- | :--- |
| DOM | 将整个xml文件读入内存，建立一个dom树来解析每个结点 | a、由于整棵树在内存中，因此可以对xml文档随机访问 b、可以对xml文档进行修改操作 c、较sax，dom使用也更简单。 | a、整个文档必须一次性解析完 a、由于整个文档都需要载入内存，对于大文档成本高 |
| SAX | 部分解析，基于事件驱动，可以注册自己感兴趣的事件，比如 EntityResolver, DTDHandler, ContentHandler, ErrorHandler接口，分别用于监听解析实体事件、DTD处理事件、正文处理事件和处理出错事件，默认实现类为DefaultHandler | a、无需将整个xml文档载入内存，因此消耗内存少 b、可以注册多个ContentHandler | a、不能随机的访问xml中的节点 b、不能修改文档 |
| JDOM | 纯java，API大量使用了Collection类，且仅使用具体类而不使用接口，自身不包含解析器，它通常使用SAX2解析器来解析和验证输入xml文件，也可以将以前构造的DOM表示作为输入，包含转换器可以将JDOM表示输出为SAX2事件流、DOM模型或XML文本文档 | a、DOM方式的优点 b、具有SAX的Java规则 | a、DOM方式的缺点 |
| DOM4 | 目前在xml解析方面是最优秀的\(Hibernate、Sun的JAXM也都使用dom4j来解析XML\)，它合并了许多超出基本 XML 文档表示的功能，包括集成的 XPath 支持、XML Schema 支持以及用于大文档或流化文档的基于事件的处理 | 最优秀的一个，集易用和性能于一身。 |  |

XPath 是一门在 XML 文档中查找信息的语言， 可用来在 XML 文档中对元素和属性进行遍历。XPath 是 W3C XSLT 标准的主要元素，并且 XQuery 和 XPointer 同时被构建于 XPath 表达之上。因此，对 XPath 的理解是很多高级 XML 应用的基础。XPath非常类似对数据库操作的SQL语言，或者说JQuery，它可以方便开发者抓起文档中需要的东西。（dom4j也支持xpath）

## 你在项目中用到了xml技术的哪些方面？如何实现的？

用到了数据存储，信息配置两方面，在做数据交换平台的时候，将数据组装成xml文件，然后将xml文件压缩加密后通过网络传送给接收者，接收解密与解压缩后再次xml文件中还原相关信息进行处理，在做软件配置时，利用xml可以很放百年的进行，软件的各种配置都可以存储在xml文件中，比如SharedPrefences。

# XML文档定义有几种形式？它们之间有何本质区别？解析XML文档有哪几种方式？

有两种定义形式，dtd文档类型定义和SchemaXML模式；XML Schema 和DTD都用于文档验证，但二者还有一定的区别，本质区别是：Scheme本身是xml的，可以被XML解析器解析，这也是从DTD上发展Schema的根本目的。另外，XML Schema 是内容开放模型，可扩展，功能性强，而DTD可扩展性差。XML Schema 支持丰富的数据类型，而 DTD不支持元素的数据类型，对属性的类型定义也很有限。XML Schema 支持命名空间机制，而DTD不支持。XML Schema 可针对不同情况对整个XML 文档或文档局部进行验证；而 DTD缺乏这种灵活性。XML Schema 完全遵循XML规范，符合XML语法，可以和DOM结合使用，功能强大；而DTD 语法本身有自身的语法和要求，难以学习。

有DOM文档对象模型，SAX（Simple API for XML）,STAX等。DOM：文档驱动，处理大行文件时，其性能下降的非常厉害，这个问题是由DOM的树结构所造成的，这种结构占用的内存较多，而且DOM必须在解析文件之前把整个文档装入内存，适合对XML的随机访问。SAX:不同于DOM，SAX是事件驱动型的XML解析方式。他顺序读取XML文件，不需要一次全部装在整个文件，当遇到像文件开头，文档结束，或者标签开头与标签结束时，他会触发一个事件，用户通过在其回调事件中写入处理代码来处理XML文件，适合对XML的顺序访问，且是只读的。当前浏览器不支持SAXSAXParserFactory factory = SAXParserFactory.newinstance\(\);SAXParser saxparser = factory.newSAXParser\(\); //创建SAX解析器MyHandler handler = new MyHandler\(\); //创建事件处理器saxParser.parse\(new File\("Sax\_1.xml"\),handler\); //绑定文件和事件处理者

STAX:Streaming API for XML ,是用“JavaTM”语言处理XML的最新标准，STAX与其他方法的区别就在于应用程序能够把XML作为一个文件流来处理。STAX允许应用程序把代码这些事件逐个拉出来，而不用提供在解析器方便时从解析器中接受事件的处理程序。

