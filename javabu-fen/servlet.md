## Servlet的基本架构

Servlet是Server与Applet的缩写，是服务端小程序的意思。是SUN公司提供的一门用于开发动态Web资源的技术。 Servlet本质上也是Java类，但要遵循Servlet规范进行编写，没有main\(\)方法，它的创建、使用、销毁都由Servlet容器进行管理\(如Tomcat\)。Servlet是和HTTP协议是紧密联系的，其可以处理HTTP协议相关的所有内容。这也是Servlet应用广泛的原因之一。提供了Servlet功能的服务器，叫做Servlet容器，其常见容器有很多，如Tomcat, Jetty, resin, Oracle Application server, WebLogic Server, Glassfish, Websphere, JBoss等。

```
public class ServletName extends HttpServlet {
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
    }
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
    }
}
```

[Servlet详解](http://www.cnblogs.com/rocomp/p/4808924.html)

##  Servlet的生命周期，以及和CGI的区别

Servlet被服务器实例化后，容器运行其init方法，请求到达时运行其Service方法，Service方法自动派遣运行与请求对应的doXXXfangfa \(doget, doPost\)等，当服务器决定将实例销毁的时候调用其Destory方法。

与CGI的区别：Servlet处于服务器进程中，它通过多线程方式运行其Service方法，一个实例可以服务于多个请求，并且其实例一般不会销毁，而CGI对每个请求都产生新的进程，服务完成后就销毁，搜易效率上低于CGI。

