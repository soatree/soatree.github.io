---
title: JavaWeb学习-tomcat和servlet
categories:
  - java
tags:
  - web
date: 2022-12-29 20:45:48
---

# 前言

本文介绍javaweb的基本知识，包括CS和BS的架构形式、web容器、servlet等。

# CS和BS

## CS：客户端服务器架构模式

优点：充分利用客户端机器的资源，减轻服务器的负荷。
缺点：需要安装；升级维护成本较高。

## BS：浏览器服务器架构模式

优点：客户端不需要安装；维护成本较低。
缺点：所有的计算和存储任务都是放在服务器端的，服务器的负荷较重；在服务端计算完成之后把结果再传输给客户端，网络负荷较重。

# Tomcat

Tomcat简单的说就是一个运行JAVA的网络服务器，底层是Socket的一个程序，它也是JSP和Serlvet的一个容器。
将编译好的java文件和前端文件按照一定格式放入Tomcat的webapps文件夹，然后启动Tomcat即可启动这个javaweb服务。 

# servlet

通过前端页面的表单中的action和method以及web项目中web.xml配置文件，可以将表单中的数据发送给后端指定的servlet，通过在servlet中实现doPost、doGet等方法可以获取前端表单的数据，可以实现前后端的数据交互。

## servlet基本信息

1. 继承关系：HttpServlet -> GenericServlet -> Servlet
2. Servlet中的核心方法： init() , service() , destroy()
3. 服务方法：当有请求过来时，service方法会自动响应（其实是tomcat容器调用的），在HttpServlet中我们会去分析请求的方式：到底是get、post、head还是delete等等，然后再决定调用的是哪个do开头的方法。

## Servlet的生命周期

1. 生命周期：从出生到死亡的过程就是生命周期。对应Servlet中的三个方法：init(),service(),destroy()。
2. 默认情况下：
  第一次接收请求时，这个Servlet会进行实例化(调用构造方法)、初始化(调用init())、然后服务(调用service())
  从第二次请求开始，每一次都是服务
  当容器关闭时，其中的所有的servlet实例会被销毁，调用销毁方法
3. 通过案例我们发现：
  - Servlet实例tomcat只会创建一个，所有的请求都是这个实例去响应。
  - 默认情况下，第一次请求时，tomcat才会去实例化，初始化，然后再服务。
  - 因此得出结论：如果需要提高系统的启动速度，当前默认情况就是这样。如果需要提高响应速度，我们应该设置Servlet的初始化时机。
4. Servlet在容器中是：单例的、线程不安全的

## html界面如何把信息传给servlet

web项目在tomcat中部署后，通过操作前端界面即可将前端界面的信息传给后端，例如以下一个html界面`http://localhost:8080/pro07/add.html`中有一个表单，当我们填写完信息点击submit按钮是，会触发add方法

```
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <form action="add" method="post">
        名称：<input type="text" name="fname"/><br/>
        价格：<input type="text" name="price"/><br/>
        库存：<input type="text" name="fcount"/><br/>
        备注：<input type="text" name="remark"/><br/>
        <input type="submit" value="添加" />
    </form>
</body>
</html>
```

此时地址栏的url变为`http://localhost:8080/pro07/add`，基于tomcat的web.xml配置，`/add`将匹配名为`AddServlet`的servlet，然后这个servlet会匹配`com.atguigu.servlets.AddServlet`类，根据add的方法为post，将匹配`com.atguigu.servlets.AddServlet`中的doPost方法并执行。

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <servlet>
        <servlet-name>AddServlet</servlet-name>
        <servlet-class>com.atguigu.servlets.AddServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>AddServlet</servlet-name>
        <url-pattern>/add</url-pattern>
    </servlet-mapping>
    <!--
    1. 用户发请求，action=add
    2. 项目中，web.xml中找到url-pattern = /add   -> 第12行
    3. 找第11行的servlet-name = AddServlet
    4. 找和servlet-mapping中servlet-name一致的servlet ， 找到第7行
    5. 找第8行的servlet-class -> com.atguigu.servlets.AddServlet
    6. 用户发送的是post请求（method=post） ， 因此 tomcat会执行AddServlet中的doPost方法
    -->
</web-app>
```

## 会话

1. Http是无状态的
  - HTTP 无状态：服务器无法判断这两次请求是同一个客户端发过来的，还是不同的客户端发过来的
  - 无状态带来的现实问题：第一次请求是添加商品到购物车，第二次请求是结账；如果这两次请求服务器无法区分是同一个用户的，那么就会导致混乱
  - 通过会话跟踪技术来解决无状态的问题。

2. 会话跟踪技术
  - 客户端第一次发请求给服务器，服务器获取session，获取不到，则创建新的，然后响应给客户端
  - 下次客户端给服务器发请求时，会把sessionID带给服务器，那么服务器就能获取到了，那么服务器就判断这一次请求和上次某次请求是同一个客户端，从而能够区分开客户端
  - 常用的API：
    request.getSession() -> 获取当前的会话，没有则创建一个新的会话
    request.getSession(true) -> 效果和不带参数相同
    request.getSession(false) -> 获取当前会话，没有则返回null，不会创建新的

    session.getId() -> 获取sessionID
    session.isNew() -> 判断当前session是否是新的
    session.getMaxInactiveInterval() -> session的非激活间隔时长，默认1800秒
    session.setMaxInactiveInterval()
    session.invalidate() -> 强制性让会话立即失效
    ....

3. session保存作用域，可以在某个session中存放数据，在该session生命周期中有效
  - session保存作用域是和具体的某一个session对应的
  - 常用的API：
    void session.setAttribute(k,v)
    Object session.getAttribute(k)
    void removeAttribute(k)

## 服务器内部转发以及客户端重定向

1. 服务器内部转发 : request.getRequestDispatcher("...").forward(request,response);
  - 一次请求响应的过程，对于客户端而言，内部经过了多少次转发，客户端是不知道的
  - 地址栏没有变化
2. 客户端重定向： response.sendRedirect("....");
  - 两次请求响应的过程。客户端肯定知道请求URL有变化
  - 地址栏有变化


## Thymeleaf - 视图模板技术

thymeleaf直接在html中使用，是一个模板引擎，可以将后端的结果动态渲染成前端界面。

1. 添加thymeleaf的jar包
2. 新建一个Servlet类ViewBaseServlet
3. 在web.xml文件中添加配置
    - 配置前缀 view-prefix
    - 配置后缀 view-suffix
4. 使得我们的Servlet继承ViewBaseServlet
5. 根据逻辑视图名称 得到 物理视图名称
//此处的视图名称是 index
//那么thymeleaf会将这个 逻辑视图名称 对应到 物理视图 名称上去
//逻辑视图名称 ：   index
//物理视图名称 ：   view-prefix + 逻辑视图名称 + view-suffix
//所以真实的视图名称是：      /       index       .html
super.processTemplate("index",request,response);
6. 使用thymeleaf的标签
  th:if   ,  th:unless   , th:each   ,   th:text

以下是thymeleaf的样例

```
//Servlet从3.0版本开始支持注解方式的注册,不用在web.xml配置表单的action和servlet的关系，直接基于注解匹配
@WebServlet("/index")
public class IndexServlet extends ViewBaseServlet {
    @Override
    public void doGet(HttpServletRequest request , HttpServletResponse response)throws IOException, ServletException {
        FruitDAO fruitDAO = new FruitDAOImpl();
        List<Fruit> fruitList = fruitDAO.getFruitList();
        //保存到session作用域
        HttpSession session = request.getSession() ;
        session.setAttribute("fruitList",fruitList);
        //此处的视图名称是 index
        //那么thymeleaf会将这个 逻辑视图名称 对应到 物理视图 名称上去
        //逻辑视图名称 ：   index
        //物理视图名称 ：   view-prefix + 逻辑视图名称 + view-suffix
        //所以真实的视图名称是：      /       index       .html
        super.processTemplate("index",request,response);
    }
}
```

以下是web.xml中的配置

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    <!-- 配置上下文参数 -->
    <context-param>
        <param-name>view-prefix</param-name>
        <param-value>/</param-value>
    </context-param>
    <context-param>
        <param-name>view-suffix</param-name>
        <param-value>.html</param-value>
    </context-param>
</web-app>
```

以下是加入thymeleaf的html，实现了后端数据的前端渲染

```
<html xmlns:th="http://www.thymeleaf.org">
	<head>
		<meta charset="utf-8">
		<link rel="stylesheet" href="css/index.css">
	</head>
	<body>
		<div id="div_container">
			<div id="div_fruit_list">
				<p class="center f30">欢迎使用水果库存后台管理系统</p>
				<table id="tbl_fruit">
					<tr>
						<th class="w20">名称</th>
						<th class="w20">单价</th>
						<th class="w20">库存</th>
						<th>操作</th>
					</tr>
					<tr th:if="${#lists.isEmpty(session.fruitList)}">
						<td colspan="4">对不起，库存为空！</td>
					</tr>
					<tr th:unless="${#lists.isEmpty(session.fruitList)}" th:each="fruit : ${session.fruitList}">
						<td th:text="${fruit.fname}">苹果</td>
						<td th:text="${fruit.price}">5</td>
						<td th:text="${fruit.fcount}">20</td>
						<td><img src="imgs/del.jpg" class="delImg"/></td>
					</tr>
				</table>
			</div>
		</div>
	</body>
</html>

```

## 保存作用域

原始情况下，保存作用域我们可以认为有四个： page（页面级别，现在几乎不用） , request（一次请求响应范围） , session（一次会话范围） , application（整个应用程序范围）
1. request：一次请求响应范围
2. session：一次会话范围有效
3. application：一次应用程序范围有效

# 参考

[尚硅谷javaweb教程](https://www.bilibili.com/video/BV1AS4y177xJ/)