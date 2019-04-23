# GrayLog基于Stream告警设置

官方建议基于Stream来实现告警，我们可以定义一些规则，比如Nginx日志中的status字段为500时，就放入指定Stream中，这样匹配起来效果最好。

## 创建Stream

如下图所示操作
![增加Stream](../images/graylog-add-stream.jpg)
完成的，返回列表页,找到刚才创建的规则，并点击"`Manage Rules`"
![管理Stream的Rule规则](../images/graylog-manage-rule.jpg)
增加路由规则
![添加规则](../images/graylog-add-rule.jpg)
下面示例增加匹配`404`的规则
![404规则](../images/graylog-404-rule.jpg)

Type可选项有多个，具体请查看官方文档。

这里的意义是，如果接收到的消息的`status`列的值匹配正则`404`,则会进入该Stream。

![完成配置](../images/graylog-rule-done.jpg)

## 创建AlertCondition--告警条件

回到`Stream`列表页，点击`Manager Alerts`
![列表页](../images/graylog-manage-rule.jpg)

创建告警规则
![告警规则创建](../images/graylog-add-cond-2.jpg)
![告警规则创建](../images/graylog-add-cond-3.jpg)

## 创建Notifications--告警方式

![创建告警](../images/graylog-alert-1.jpg)

![创建告警](../images/graylog-alert-2.jpg)

`Sender`的值需要同配置文件的发送邮件地址一样
`E-Mail Receviers`是接收人，直接在输入框输入后添加

相关内容填好保存起来

下面是都配置好的截图
![完整截图](../images/graylog-alert-done.jpg)
可以点击`TEST`来进行测试配置是否正确