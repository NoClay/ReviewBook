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

