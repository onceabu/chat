# Chat
一个社交APP
# 概述
整体实现是按照传统的Android+Servlet+Json+MySql方案进行的，实时聊天借助Socket在服务端建立队列连接并通信，动态的请求和发送借助Servlet+Json实现，Android方面使用了Okhttp、Glide、circleimageview一些常用的框架，具体实现将在下面说明。（主要在Android方面的实现，数据库和服务端不做过多说明）
# 数据库
数据库的设计包含用户信息、用户关系、动态、评论、消息、二次评论和赞，具体直接上图<br>
![](https://github.com/onceabu/chat/raw/master/picture/database.jpg)<br>
图中同种颜色为主外键关系，粗线框为主键，细线框为外键，其他细节这里不再过多描述。要说明的一点就是图片的存储在数据库中只是索引字符，发布动态后服务端将照片读取并存储到文件夹中，Android端从数据库获取索引后借助Glide加载图片。
# 服务端
服务端Tomcat+MyEclipse，具体的数据获取和发送以及逻辑处理这里不过多描述，主要说明一下实时聊天的实现，在服务启动时开启一条子线程，该线程主要负责通信队列的建立和管理，在Android端登录时会与服务端建立Socket连接并以ID为索引加入队列中。
```
...
mSS=new ServerSocket(30000);
while(true){
	Socket socket=mSS.accept();
	bufferedReader=new BufferedReader(new InputStreamReader(s.getInputStream(),"utf-8"));
	try{
		String ID=bufferedReader.readLine();
		if(ID!=null){
			socketList.put(ID,socket);
			new Thread(new ServerThread(socket)).start();
		}
	}catch(IOException e){
		e.printStackTrace();
	}
}
...
```
ServerThread主要负责对消息的处理，当用户A向用户B发送一条消息，ServerThread会接收相关的ID以及消息内容，然后在通信队列中查找用户B的Socket连接，如果存在直接向其发送消息，如果不存在说明用户B不在线，此时存储消息等待用户B上线后再次获取，当用户退出应用时Socket连接也会断开退出通信队列。
# Android端
功能展示只有截图和gif，但截图只能展示页面布局，gif受大小限制也只是一些主要的东西，整体详细情况可通过录屏video获知，这里不做上传。
## 一、整体结构
应用主要包括登录与注册、好友动态、寻找好友、个人信息和好友聊天五个模块，具体结构直接上图。
![](https://github.com/onceabu/chat/raw/master/picture/dxc.png)
在第一次进入应用时选择登录或注册，在成功登录后进入主页面，主页面构成为ViewPager+Fragment+自定义选项菜单。自定义选项菜单以线性布局结合相对布局，与ViewPager联动，在主页面初始化时与服务端连接从数据库获取未读消息或者好友请求等信息数，信息数在自定义选项菜单中显示，Fragment包含好友动态、寻找好友、个人信息和好友聊天，整体预览图如下。
![](https://github.com/onceabu/chat/raw/master/picture/main.gif)
## 二、登录与注册
* UI<br>
登录界面布局方面以线性布局结合相对布局，邮箱输入框基于AutoCompleteTextView的自定义控件，通过监听自动完成邮箱地址的拓展，左侧CompoundDrawables根据焦点高亮显示，右侧CompoundDrawables根据Touch事件的坐标判断进行内容的删除。
注册界面以帧布局为主，邮箱和密码的输入完成后对邮箱是否重复和密码格式的准确性进行判断，符合后隐藏邮箱密码输入页面，显示个人信息输入页面
![](https://github.com/onceabu/chat/raw/master/picture/lasa.png)<br>
* 功能实现<br>
注册中对邮箱、密码、手机号等不同格式的信息输入通过正则表达式进行判断，同时也可以通过图片选择器上传头像和个人主页背景，关于图片选择器的具体实现会在好友动态中详细说明。注册过程中字符和图片数据通过OkHttp发送至服务端并存储到mysql用户表中，同样图片文件存储到文件夹中，数据库只保存图片的字符索引。注册成功后借助sharePreference将用户的相关信息保存在本地，便于以后登录的直接使用。
## 三、好友动态


