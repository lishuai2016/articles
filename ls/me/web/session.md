# session

书中讲：
以下情况，Session结束生命周期，Servlet容器将Session所占资源释放：
1.客户端关闭浏览器
2.Session过期
3.服务器端调用了HttpSession的invalidate()方法。
"一个浏览器就是一个新session,关了浏览器session就结束了"
session 是在服务器端建立的，浏览器访问服务器会有一个sessionid，浏览器端通过sessionid定位服务器端的session,session的创建和销毁由服务器端控制。当浏览器关闭后，session还存在在服务器端，只不过你新开的浏览器去访问服务器会创建另一个session,这个时候的 sessionid已经不一样了。也就不能访问上一次的哪个session里面的内容了。 
"session的创建和销毁由服务器端控制",服务器端才有session,客户端只是通过sessionid来匹配session.
那服务器端session如何建的呢？ 普通html不会创建，jsp默认是创建的，只要你访问任何一个jsp就会创建（不过只创建一次），你关闭浏览器重新访问又会创建一个，这些创建的session由服务器自己控制销毁，你也可以在服务器端代码中销毁。

什么情况下需要用上这种服务器端的session方式？ 
默认情况下，jsp被访问就会创建session(最开始是空的没有数据的),你的应用中的代码只是往session里面put数据。网上说可以 通过 ＜%@ page session="false"%＞来不让jsp自动创session.我自己测试了一下（用sessionlistener），根本不起作用， session照样创建成功。
最后说一下，只有服务器端才有session.客户端被存到本地的是cookie.不过安全性低。所以不能放重要的数据。

============================================================================

sesion其实简单:
先request.getsession()，当已有一个session与前request相关时就返回对这个 session的引用，当没有时就生成一个.一个session在server通过一个sessionid来标识的。也就是说在一个server是不会有 两个相同sessionid的session.
那麼session为什麼会和cookie扯在一起呢?
正如我所说对於一 个session来说它的sessionid就是其身份的标识。若我们将这个sessionid保存到用户端，当同一个会话的后序请求来时都将这个 sessionid放在request 的header中(也就是我们说的cookie)这样不就可以来验证这个request是否与之前的request是同一个会话了吗!
什麼是会话呢?
我 们可以通俗一点理解。只要你的browers不关我们就称这一系列的request与response为一个会话。一断你close就称这个会话已结束。 虽然会话结束但并不代表你的session就被destroy.因为session是存活在server上的。它的生命完全由server来主宰 （web.xml中的设定）.
虽然你的session还存活在server上但你已无法再取得它。因为j2ee的api只给我们一种方法来取得与当前会话相关的session的引用:request.getsession() or reqeust.getsession(boolean)
=======================================================================

一个常见的误解是以为session在有客户端访问时就被创建，然而事实是直到某server端程序调用HttpServletRequest.getSession(true)这样的语句时才被创建，注意如果JSP没有显示的使用 <%@page session="false"%> 关闭session，则JSP文件在编译成Servlet时将会自动加上这样一条语句HttpSession session = HttpServletRequest.getSession(true);
这也是JSP中隐含的session对象的来历。
<**********************有点矛盾的地方,到底JSP显示的使用< %@page session="false"%>能不能让服务端不创建sessionid呢？试验下**************************&gt;
＜%@ page session="false"%＞ 
不是不让页面创建Session,而是在此JSP页面无法使用session.可以减少网络数据传输.

例子：使得session失效

@RequestMapping("logout.do")
     public String logout(ModelMap model,HttpServletRequest request,HttpServletResponse response){
          String returnUrl = request.getParameter("returnUrl");
          HttpSession session = request.getSession(false);
          if (session != null) {
              session.invalidate();
          }
          return "redirect:http://cas.zfwx.com/logout?service="+returnUrl;
     }
session对象的销毁时机　　session对象默认30分钟没有使用，则服务器会自动销毁session，在web.xml文件中可以手工配置session的失效时间，例如：
 1 <?xml version="1.0" encoding="UTF-8"?>
 2 <web-app version="2.5" 
 3     xmlns="http://java.sun.com/xml/ns/javaee" 
 4     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
 5     xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
 6     http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"> 7   <display-name></display-name>
 8   
 9   <welcome-file-list>
10     <welcome-file>index.jsp</welcome-file>
11   </welcome-file-list>
12 
13   <!-- 设置Session的有效时间:以分钟为单位-->
14     <session-config>
15         <session-timeout>15</session-timeout>
16     </session-config>17 
18 </web-app>

　　当需要在程序中手动设置Session失效时，可以手工调用session.invalidate方法，摧毁session。
1 HttpSession session = request.getSession();
2 //手工调用session.invalidate方法，摧毁session3 session.invalidate();
