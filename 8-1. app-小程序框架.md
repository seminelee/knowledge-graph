# wepy

wepy：vue+webpack

功能：脚手架、编译打包、核心库

编译原理：

1. index.wpy拆解成style、template、script，script中拆解出json文件
2. 编译前梳理组件引用关系与npm引用关系
3. 使用各种loader编译成wxss、wxml、js文件
4. 各种插件处理，代码混淆插件、wxml/图片压缩插件
5. 最终生成wxss、wxml、js、json

缺点：

1. 语法解析：编译`template`用的是xmldom，这个库已经没有人维护了等等，报错时无法很快定位是哪个地方报错了。
2. 类Vue语法，某些Vue的API不支持
3. 错误处理机制：报错没有详细的信息（位置等），比较难快速定位问题
4. 数据绑定性能优化：解决频繁`setData`的问题。diff方法的优化。

v2.0数据绑定优化： 

- v1.0
  初始化时对树进行深拷贝->当有修改时与拷贝的树相比较->得出脏数据->`setData`
- v2.0
  参考Vue并做了优化
  初始化时创建watcher->当有修改时watcher会记录修改（key-path-value），并将脏数据放到队列里->`setData` （不会做深比较了）



# QQ互联



QQ登录OAuth2.0采用OAuth2.0标准协议来进行用户身份验证和获取用户授权，相对于之前的[OAuth1.0协议](http://wiki.connect.qq.com/qzone_oauth_1-0认证简介)，其认证流程更简单和安全。



QQ互联升级原因：历史原因，QQ对公司内业务登录及查询服务均以uin(QQ号)为关键字直接暴露，且存在业务方将接口二次包装对外提供服务的行为，对QQ用户的安全性存在暴露的风险；旧的cookie中domain设置的是*.qq.com，各网站登录态不独立。



改造：

1. 数据库中的QQ号转为openid，申请接口转换
2. 接入QQ互联，写好重定向逻辑，根据restful设计规范，接口统一用/v2前缀，兼容旧逻辑，用户无感应地接入。
3. 不再获取用户QQ号，而是获取用户openid，不再手动给用户QQ号授权，开发用户中心让用户自己添加QQ号，将openid存入数据库中

4. 全部系统接入新登录鉴权后，删除旧逻辑



# hippy

![image-20191205155603979](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205155603979.png)

最终是native渲染

## RN Weex 对比

RN的缺点：包太大、启动慢、list性能差；license问题 协议风险。Airbnb宣布放弃

![image-20191205155939379](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205155939379.png)

首屏信息流业务 数千万DAU，发布系统

## 架构设计

![image-20191205160538057](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160538057.png)

![x x x](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160319004.png)

![image-20191205160339070](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160339070.png)

安卓的优化：一个引擎里多个JSContext，即启动快，又将业务隔离，避免污染。

包体积小，APK安装包小。

![image-20191205160619934](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160619934.png)

![image-20191205160829161](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160829161.png)

核心就是一个渲染引擎。类比浏览器的渲染引擎。

![image-20191205160952480](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205160952480.png)

## 比RN好的地方

### 手势

设计最重的是手势事件这一块

RN：

![image-20191205161249000](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205161249000.png)![image-20191205161310213](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205161310213.png)

![image-20191205161427323](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205161427323.png)

### 通信

RN5毫秒缓存

hippy：终端调前端通过Bridge里的函数，前端调终端也是通过Bridge的函数直接通信

### 动画

RN：终端只是计时的作用，终端每隔一段时间就告诉前端setState。前端通过消息队列将动画事件传给终端。队列里有渲染的事件也有动画的事件，渲染的事件一多就会掉帧。![image-20191205162031917](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205162031917.png)

hippy：前端一开始讲整个动画事件传给终端，由终端驱动这个动画。只有一次通信。

![image-20191205162237656](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205162237656.png)



### 终端的优化

首屏启动更快。开启了V8的XXcache

![image-20191205163526733](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205163526733.png)

listview的优化

终端构建了虚拟dom，优化diff算法。对list里的每个item做了复用。

![image-20191205171616376](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171616376.png)

![image-20191205171752566](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171752566.png)

so加载是什么？

![image-20191205163918518](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205163918518.png) 



### 三端同构

![image-20191205164254155](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205164254155.png)

![image-20191205164435316](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205164435316.png)



### 发布和运营

有发布管理系统（打包、发布）

动态运营服务（App隔离、灰度、ABTest、差量包、安全校验、强制更新、流控、push）



# feeds实践

![image-20191205170419851](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205170419851.png)

减少jsbundle

eslint、图片压缩、npm包减少滥用

hippyCallNatives中打日志

![image-20191205171035413](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171035413.png)

优化：减少层级结构、首屏只渲染可见的、渲染完tab数据再拉取每个tab的数据

![image-20191205171324319](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171324319.png)

![image-20191205171512445](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171512445.png)



listview 优化![image-20191205171923410](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205171923410.png)

![image-20191205172042566](/Users/lixiaomei/Library/Application Support/typora-user-images/image-20191205172042566.png)