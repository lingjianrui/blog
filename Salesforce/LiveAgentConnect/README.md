
# LiveAgentConnect

![image](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect.gif)

[TOC]

# LiveAgentConnect

## 开发

### 代码结构
![title](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-1.jpg)

### 程序设计
WechatConnectEndpoint.java

 - 用于接收wechat的推送消息
 - 443 端口可在配置文件中配置
 - 调用了connect服务

Connect.java

 - Connect服务处理主要的业务逻辑 处于业务逻辑层
 - Connect调用了LiveAgentService
   -调用LiveAgent SessionId API 获取SESSION-ID和SESSION-KEY，AFFINITY
   -调用LiveAgent ChasitorInit 初始化Session
   -调用LiveAgent Message 方法 从LiveAgent 里获取消息
   -调用LiveAgent ChatMessage 方法 将消息发送到LiveAgent
 - Connect 调用了WechatService
   -调用了Wechat GetAccessToken 获取token
   -调用了Wechat 发送消息到wechat
   -调用了Wechat 获取联系人详细信息

LiveAgentService.java

 - 提供获取sessionid方法
 - 提供初始化session的方法
 - 提供收信息
 - 提供发信息

WechatService.java

 - 提供获取AccessToken的方法
 - 提供wechat 收信息
 - 提供wechat 发信息

ScheduledTasks.java

 - 访问微信资源需要 微信的AccessToken
 - 这个程序 会根据配置定时更新AccessToken到mongodb 中去

### 配置说明
配置文件src/main/resources/application.properties
设置服务端口号也可以用--server.port=443在命令行后面修改该参数
server.port = 8443
server.ssl.key-store = classpath:sample.jks
server.ssl.key-store-password = secret
server.ssl.key-password = password 

**Mongodb配置** 
请不要试用test库来做生产环境。test库是用来单元测试的
spring.data.mongodb.host  mongodb 数据库所在服务器ip或domain 默认localhost
spring.data.mongodb.port  mongodb 数据库默认端口 27017
spring.data.mongodb.database mongodb 数据库数据库名称 test
logging.path log位置设置
access_token_refresh_interval.setup  设置微信access_token刷新时间 默认3600000 

**LiveAgent配置** 
live_agent.endpoint [LiveAgentEnpoint设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-2.jpg)
live_agent.deployment_id [Deployment_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-3.jpg)
live_agent.org_id [Org_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-4.jpg)
live_agent.button_id [Button_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-5.jpg)
live_agent.api.version=37
live_agent.nickname.prefix=微信: 

**Salesforce配置**
salesforce.clientid [ClientID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-6.jpg)
salesforce.clientsecret [ClientSecret设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-7.jpg)
salesforce.username salesforce 用户设置
salesforce.password salesforce 用户密码+token
salesforce.restapi.version=v37.0
salesforce.sobject.name=Account 

**Wechat配置**
wechat.app_id 微信app id设置
wechat.app_secret 微信app secret 设置 

#### LiveAgentEnpoint设置
Setup -> Customize -> Live Agent -> Live Agent Setting
![LiveAgentEnpoint设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-8.jpg) 

#### Deployment_ID设置
Setup -> Customize -> Live Agent -> Deployments
![Deployment_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-9.jpg) 

#### Org_ID设置
Setup -> Company Profile -> Company Information
![Org_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-10.jpg) 

#### Button_ID设置
Setup -> Customize -> Live Agent -> ChatButtons & Invitations
![Button_ID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-11.jpg) 

#### ClientID设置
Setup -> Create -> App -> ConnectedApp
![ClientID设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-12.jpg) 

#### ClientSecret设置
Setup -> Create -> App -> ConnectedApp
![ClientSecret设置](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-13.jpg) 

## 测试
使用HttpClient模拟微信推送 进行本地测试。 
请求细节如下  
1.确保mongodb数据库服务已经启动 27017端口已经开启  
2.在eclipse中LiveAgentConnect程序  
![在eclipse中LiveAgentConnect程序1](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-14.jpg) 
![在eclipse中LiveAgentConnect程序2](https://github.com/lingjianrui/blog/blob/master/images/LiveAgentConnect-15.jpg) 
3.模拟微信推送请求 
POST https://localhost:8443/sfdc/msg 
```
<xml>
    <ToUserName>xiaoheiaa</ToUserName>
    <FromUserName>onw0Tt_fbdqQNw52EmOf4FDpP_64</FromUserName>
    <CreateTime>1348831860</CreateTime>
    <MsgType>text</MsgType>
    <Content>你hao</Content>
    <MsgId>1234567890123456</MsgId>
</xml>
```
4.如果eclipse console里面没有反映
请使用浏览器发送请求https://localhost:8443/sfdc/test 测试service是否运行正常
然后再尝试模拟微信推送。

## 部署
1.从github上下载下来源代码后 在代码根目录执行 mvn clean package 
2.在target目录下执行(如下的这种执行方式会覆盖配置文件中的定义)
java -jar liveagent.connect-0.0.1-SNAPSHOT.jar \
--live_agent.endpoint=xxxx.salesforceliveagent.com \
--live_agent.deployment_id=xxxxxxxx \
--live_agent.org_id=xxxxxxxx \
--live_agent.button_id=xxxxxxx \
--logging.path=/var/log \
--server.port=443
3.确保你的服务器已经在微信公众号中配置成功。(能收到微信推送过来的消息)
4.确保你的mongodb服务器运行正常

