# Chat
一个社交APP
# 概述
　　很早就有敲个社交应用的想法，种种原因一直没有付诸行动，还好最终还是做起来了。<br>
　　整体实现是按照传统的Android+Servlet+Json+MySql方案进行的，实时聊天借助Socket在服务端建立队列连接并通信，动态的请求和发送借助Servlet+Json实现，数据库的设计以及客户端的实现将在下面具体说明。
# 数据库
　　数据库实现并不复杂，包含用户信息、用户关系、动态、评论、消息、二次评论和赞，具体直接上图
![](https://github.com/onceabu/chat/raw/master/picture/database.jpg)
