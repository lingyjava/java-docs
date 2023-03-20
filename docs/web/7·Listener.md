# 7·Listener

- [7·Listener](#7listener)
  - [Listener是什么](#listener是什么)
  - [使用方式](#使用方式)

## Listener是什么
Listener（监听器），在Servlet中主要用于对Context、request、session对象的生命周期进行监控。
使用监听器可以对用户的更多操作做出反应。

## 使用方式
1. 使一个类重写valueBound()和valueUnbound()方法。
```java
//设置一个num,代表当前在线用户数

//使用监听器需要将类实现一个可序列化接口(空接口)
public class User implements HttpSessionBindingListener {
    @Override
    public void valueBound(HttpSessionBindingEvent httpSessionBindingEvent) {
        //当把该user对象放到session中的时候会触发该方法
        ServletContext app=httpSessionBindingEvent.getSession().getServletContext();
        Integer num  = (Integer) app.getAttribute("num");
        if (num == null) {
            num = 1;
        }else{
            num++;
        }
        app.setAttribute("num",num);

    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent httpSessionBindingEvent) {
        //当该user从session中移除,或者session对象销毁的时候触发该方法
        ServletContext app=httpSessionBindingEvent.getSession().getServletContext();
        Integer num=(Integer) app.getAttribute("num");
        num--;
        app.setAttribute("num", num);
    }
}

```
