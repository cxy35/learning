## 公众号
### 模板消息
- 介绍：模板消息仅用于公众号向用户发送重要的服务通知，只能用于符合其要求的服务场景中，如信用卡刷卡通知，商品购买成功通知等。不支持广告等营销类消息以及其它所有可能对用户造成骚扰的消息。
- 要求：服务号且完成微信认证（个人号无法认证）。
- 文档：https://developers.weixin.qq.com/doc/offiaccount/Message_Management/Template_Message_Interface.html


## 小程序
### 订阅消息
- 介绍：消息能力是小程序能力中的重要组成，我们为开发者提供了订阅消息能力，以便实现服务的闭环和更优的体验。
  - 订阅消息推送位置：服务通知。
  - 订阅消息下发条件：用户自主订阅。
  - 订阅消息卡片跳转能力：点击查看详情可跳转至该小程序的页面。
- 要求：个人号无法新增模板，可以基于某个公共模板去申请关键字。
- 类型：一次性订阅消息、长期订阅消息（目前长期性订阅消息仅向政务民生、医疗、交通、金融、教育等线下公共服务开放，后期将逐步支持到其他线下公共服务业务。）、设备订阅消息。
- 文档：https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/subscribe-message.html
### 客服消息


<Resources>
	<PreResources className="org.apache.catalina.webresources.FileResourceSet" base="${catalina.base}/webapps/stm-web/WEB-INF/lib/jackson-core-2.8.10.jar" webAppMount="/WEB-INF/lib/jackson-core-2.8.10.jar" />
	<PreResources className="org.apache.catalina.webresources.FileResourceSet" base="${catalina.base}/webapps/stm-web/WEB-INF/lib/jackson-annotations-2.8.0.jar" webAppMount="/WEB-INF/lib/jackson-annotations-2.8.0.jar" />
	<PreResources className="org.apache.catalina.webresources.FileResourceSet" base="${catalina.base}/webapps/stm-web/WEB-INF/lib/jackson-databind-2.8.10.jar" webAppMount="/WEB-INF/lib/jackson-databind-2.8.10.jar" />
</Resources>


<Resources>
	<PreResources className="org.apache.catalina.webresources.DirResourceSet" base="SomePath\External-lib\" webAppMount="/WEB-INF/lib" />
</Resources>


【西湖区基层治理四平台】：
1.问题描述：完成【浙江省数字政府统一用户中心单点登录】功能开发后，生产环境中业务系统无法正常启动。
2.问题分析：依据【浙江省数字政府统一用户中心单点登录集成.docx】文档，选择 Java 插件式集成时，使用到了省里提供的【JWT-SDK-1.1.1_1.8.jar】开发包，通过查看源码发现，该开发包中包含了第三方（jackson）某个老版本的源码，但包路径未做修改。同时，业务系统原本也使用了第三方（jackson）另一个新版本的 jar 包，这就导致了部分 Java 类功能不一致，最终造成系统启动异常。
3.问题解决：业务系统优化 jar 包加载顺序，优先使用业务系统原本新版本的 jar 包，避免冲突。
4.另建议：类似【JWT-SDK-1.1.1_1.8.jar】这种自研开发包，如果内部使用了第三方的源码，建议修改一下包名，避免业务系统使用时造成冲突。