---
title: session
categories: 
- http
tags:
---

使用session保存登录信息
登录成功后会在用户的浏览器的中设置一个cookie，如下所示
JSESSIONID	7CC1F0D13C81A3C5F02DE09
用户再次访问应该的session对象的id=7CC1F0D13C81A3C5F02DE09
不同的用过登录，在服务器后台会保存这些session的信息


session机制。session机制是一种服务器端的机制，服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。 

　　当程序需要为某个客户端的请求创建一个session时，服务器首先检查这个客户端的请求里是否已包含了一个session标识------------称为session id，如果已包含则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（检索不到，会新建一个），如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次响应中返回给客户端保存。


        保存这个session id的方式可以采用cookie，这样在交互过程中浏览器可以自动的按照规则把这个标识发挥给服务器。一般这个cookie的名字都是类似于SEEESIONID。但cookie可以被人为的禁止，则必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。
       经常被使用的一种技术叫做URL重写，就是把session id直接附加在URL路径的后面。还有一种技术叫做表单隐藏字段。就是服务器会自动修改表单，添加一个隐藏字段，以便在表单提交时能够把session id传递回服务器。

 

Jsessionid?

　　Jsessionid只是tomcat的对sessionid的叫法，其实就是sessionid；在其它的容器也许就不叫jsessionid了。









1、概念
指一类用来在客户端与服务器之间保持状态的解决方案
这种解决方案的存储结构

2、特点
由于 Session 是以文本文件形式存储在服务器端的，所以不怕客户端修改 Session 内容。（也可以用其他存储方式比如redis）
Session对象是有生命周期的
Session实例是轻量级的，所谓轻量级：是指他的创建和删除不需要消耗太多资源
Session对象内部有一个缓存

3、用法
Session 对象存储特定用户会话所需的属性及配置信息，在web页跳转时，信息将不会丢失，通常用于以下操作
存储整个会话过程中保持用户状态的信息，比如登录信息或者用户浏览时产生的其它信息；
存储只需要在 页重新加载 过程中，或者  一组功能页  之间保持状态的对象；
在 Web服务器上保持用户的  状态信息  供在任何时间从任何设备上的页面进行访问；

4、限制
用户登录越多，session需要的内存量越大
每个 Session 对象的持续时间是用户访问的时间加上不活动的时间。

5、为何需要session
HTTP协议本身是无状态的
举个喝咖啡的例子：
1）、该店的店员很厉害，能记住每位顾客的消费数量，只要顾客一走进咖啡店，店员就知道该怎么对待了。这种做法就是协议本身支持状态。
2）、发给顾客一张卡片，上面记录着消费的数量，一般还有个有效期限。每次消费时，如果顾客出示这张卡片，则此次消费就会与以前或以后的消费相联系起来。这种做法就是在客户端保持状态。
3）、发给顾客一张会员卡，除了卡号之外什么信息也不纪录，每次消费时，如果顾客出示该卡片，则店员在店里的纪录本上找到这个卡号对应的纪录添加一些消费信息。这种做法就是在服务器端保持状态。

6、具体机制
当程序需要为某个客户端的请求创建一个session的时候，服务器首先检查这个客户端的请求里是否已包含了一个  session标识   - 称为session id，
如果已包含一个session id则说明以前已经为此客户端创建过session，服务器就按照session id把这个session检索出来使用（如果检索不到，可能会新建一个），
如果客户端请求不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个 既不会重复，
又不容易被找到规律以仿造的字符串 ，这个session id将被在本次响应中返回给客户端保存。

由于cookie可以被人为的禁止，必须有其他机制以便在cookie被禁止时仍然能够把session id传递回服务器。经常被使用的一种技术叫做URL重写
两种形式：
// 作为url附加路径
1）、'http://..../xxx;jsessionid=abcdefjijeoijoifjioe'
// 作为查询字符串
2）、'http://..../xxx?jsessionid=abcdefjijeoijoifjioe'
较老的技术，表单隐藏字段，此方法在防止csrf中有用

7、session和Tomcat的关系
对Tomcat而言，Session是一块在服务器开辟的内存空间，其存储结构为ConcurrentHashMap；
Http协议是一种无状态协议，即每次服务端接收到客户端的请求时，都是一个全新的请求，服务器并不知道客户端的历史请求记录；
Session的主要目的就是为了弥补Http的无状态特性。简单的说，就是服务器可以利用session存储客户端在同一个会话期间的一些操作记录；

先看两个问题，如下：
1）、服务器如何判断客户端发送过来的请求是属于同一个会话？

答：用Session id区分，Session id相同的即认为是同一个会话，在Tomcat中Session id用JSESSIONID表示；

2）、服务器、客户端如何获取Session id？Session id在其之间是如何传输的呢？

答：服务器第一次接收到请求时，开辟了一块Session空间（创建了Session对象），同时生成一个Session id，并通过响应头的Set-Cookie：“JSESSIONID=XXXXXXX”命令，向客户端发送要求设置cookie的响应；

客户端收到响应后，在本机客户端设置了一个JSESSIONID=XXXXXXX的cookie信息，该cookie的过期时间为浏览器会话结束；

接下来客户端每次向同一个网站发送请求时，请求头都会带上该cookie信息（包含Session id）；

然后，服务器通过读取请求头中的Cookie信息，获取名称为JSESSIONID的值，得到此次请求的Session id；

ps：服务器只会在客户端第一次请求响应的时候，在响应头上添加Set-Cookie：“JSESSIONID=XXXXXXX”信息，接下来在同一个会话的第二第三次响应头里，是不会添加Set-Cookie：“JSESSIONID=XXXXXXX”信息的；

而客户端是会在每次请求头的cookie中带上JSESSIONID信息；
只要浏览器未关闭，在访问同一个站点的时候，其请求头Cookie中的JSESSIONID都是同一个值，被服务器认为是同一个会话。

问题：即使服务器重启了，而浏览器中的cookie的JSESSIONID没有过期（浏览器在服务器重启的过程中没有关闭），用户还在登录状态


8、Tomcat中的session实现
Tomcat中一个会话对应一个session，其实现类是StandardSession，查看源码，可以找到一个attributes成员属性，即存储session的数据结构，为ConcurrentHashMap，支持高并发的HashMap实现；

    /**
     * The collection of user data attributes associated with this Session.
     */
    protected Map<String, Object> attributes = new ConcurrentHashMap<String, Object>();
那么，tomcat中多个会话对应的session是由谁来维护的呢？ManagerBase类，查看其代码，可以发现其有一个sessions成员属性，存储着各个会话的session信息：

    /**
     * The set of currently active Sessions for this Manager, keyed by
     * session identifier.
     */
    protected Map<String, Session> sessions = new ConcurrentHashMap<String, Session>();
接下来，看一下几个重要的方法，

服务器查找Session对象的方法
客户端每次的请求，tomcat都会在HashMap中查找对应的key为JSESSIONID的Session对象是否存在，可以查看Request的doGetSession方法源码，如下源码：

protected Session doGetSession(boolean create) {

        // There cannot be a session if no context has been assigned yet
        Context context = getContext();
        if (context == null) {
            return (null);
        }

        // Return the current session if it exists and is valid
        if ((session != null) && !session.isValid()) {
            session = null;
        }
        if (session != null) {
            return (session);
        }

        // Return the requested session if it exists and is valid
        Manager manager = context.getManager();
        if (manager == null) {
            return null;        // Sessions are not supported
        }
        if (requestedSessionId != null) {
            try {
                session = manager.findSession(requestedSessionId);
            } catch (IOException e) {
                session = null;
            }
            if ((session != null) && !session.isValid()) {
                session = null;
            }
            if (session != null) {
                session.access();
                return (session);
            }
        }

        // Create a new session if requested and the response is not committed
        if (!create) {
            return (null);
        }
        if ((context != null) && (response != null) &&
            context.getServletContext().getEffectiveSessionTrackingModes().
                    contains(SessionTrackingMode.COOKIE) &&
            response.getResponse().isCommitted()) {
            throw new IllegalStateException
              (sm.getString("coyoteRequest.sessionCreateCommitted"));
        }

        // Re-use session IDs provided by the client in very limited
        // circumstances.
        String sessionId = getRequestedSessionId();
        if (requestedSessionSSL) {
            // If the session ID has been obtained from the SSL handshake then
            // use it.
        } else if (("/".equals(context.getSessionCookiePath())
                && isRequestedSessionIdFromCookie())) {
            /* This is the common(ish) use case: using the same session ID with
             * multiple web applications on the same host. Typically this is
             * used by Portlet implementations. It only works if sessions are
             * tracked via cookies. The cookie must have a path of "/" else it
             * won't be provided to for requests to all web applications.
             *
             * Any session ID provided by the client should be for a session
             * that already exists somewhere on the host. Check if the context
             * is configured for this to be confirmed.
             */
            if (context.getValidateClientProvidedNewSessionId()) {
                boolean found = false;
                for (Container container : getHost().findChildren()) {
                    Manager m = ((Context) container).getManager();
                    if (m != null) {
                        try {
                            if (m.findSession(sessionId) != null) {
                                found = true;
                                break;
                            }
                        } catch (IOException e) {
                            // Ignore. Problems with this manager will be
                            // handled elsewhere.
                        }
                    }
                }
                if (!found) {
                    sessionId = null;
                }
                sessionId = getRequestedSessionId();
            }
        } else {
            sessionId = null;
        }
        session = manager.createSession(sessionId);

        // Creating a new session cookie based on that session
        if ((session != null) && (getContext() != null)
               && getContext().getServletContext().
                       getEffectiveSessionTrackingModes().contains(
                               SessionTrackingMode.COOKIE)) {
            Cookie cookie =
                ApplicationSessionCookieConfig.createSessionCookie(
                        context, session.getIdInternal(), isSecure());

            response.addSessionCookieInternal(cookie);
        }

        if (session == null) {
            return null;
        }

        session.access();
        return session;
    }

先看doGetSession方法中的如下代码，这个一般是第一次访问的情况，即创建session对象，session的创建是调用了ManagerBase的createSession方法来实现的; 另外，注意response.addSessionCookieInternal方法，该方法的功能就是上面提到的往响应头写入“Set-Cookie”信息；最后，还要调用session.access方法记录下该session的最后访问时间，因为session是可以设置过期时间的；
session = manager.createSession(sessionId);

        // Creating a new session cookie based on that session
        if ((session != null) && (getContext() != null)
               && getContext().getServletContext().
                       getEffectiveSessionTrackingModes().contains(
                               SessionTrackingMode.COOKIE)) {
            Cookie cookie =
                ApplicationSessionCookieConfig.createSessionCookie(
                        context, session.getIdInternal(), isSecure());

            response.addSessionCookieInternal(cookie);
        }

        if (session == null) {
            return null;
        }

        session.access();
        return session;
再看doGetSession方法中的如下代码，这个一般是第二次以后访问的情况，通过ManagerBase的findSession方法查找session，其实就是利用map的key从ConcurrentHashMap中拿取对应的value，这里的key即requestedSessionId，也即JSESSIONID，同时还要调用session.access方法，记录下该session的最后访问时间；
if (requestedSessionId != null) {
            try {
                session = manager.findSession(requestedSessionId);
            } catch (IOException e) {
                session = null;
            }
            if ((session != null) && !session.isValid()) {
                session = null;
            }
            if (session != null) {
                session.access();
                return (session);
            }
        }
在session对象中查找和设置key-value的方法
这个我们一般调用getAttribute/setAttribute方法：

getAttribute方法很简单，就是根据key从map中获取value；

setAttribute方法稍微复杂点，除了设置key-value外，如果添加了一些事件监听（HttpSessionAttributeListener）的话，还要通知执行，如beforeSessionAttributeReplaced， afterSessionAttributeReplaced， beforeSessionAttributeAdded、 afterSessionAttributeAdded。。。

9、session存在的问题
安全性，session劫持，这个前面已经举过例子了；
增加服务器压力，因为session是直接存储在服务器的内存中的；
如果存在多台服务器的话，还存在session同步问题，当然如果只有一台tomcat服务器的话，也就没有session同步的事情了，然而现在一般的应用都会用到多台tomcat服务器，通过负载均衡，同一个会话有可能会被分配到不同的tomcat服务器，因此很可能出现session不一致问题；解决session同步问题，实际上主要是保证能够抽离出一块共享空间存放session信息，且这块空间不同的tomcat服务器都可以访问到；一般这块共享的空间可以是数据库，或者某台服务器的内存空间，甚至硬盘空间，或者客户端的cookie也是可以的；
或者使用Redis单点登录解决多服务器的问题。





参考：
https://www.cnblogs.com/chenpi/p/5434537.html
https://www.cnblogs.com/fnng/archive/2012/08/14/2637279.html