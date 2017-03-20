---
title: Jenkins忘记管理员密码
date: 2017-03-08 16:47:17
categories:
  - Jenkins
tags:
  - Jenkins
---

服务器部署Jenkins后，首次使用文件中的密码进入系统。
``` shell
${home}/.jenkins/secrets/initialAdminPassword
```

然后选择安装默认插件，成功后配置了一个admin账户test/test。完成后重启服务器后重新登录系统，发现无法登录了？尴了个尬！！！

求助伟大的google，找到了解决办法：
<!-- more -->
打开jenkins配置目录下的config.xml文件。
``` xml
<useSecurity>true</useSecurity>
    <authorizationStrategy class="hudson.security.FullControlOnceLoggedInAuthorizationStrategy">
    <denyAnonymousReadAccess>false</denyAnonymousReadAccess>
</authorizationStrategy>
<securityRealm class="hudson.security.HudsonPrivateSecurityRealm">
    <disableSignup>false</disableSignup>
    <enableCaptcha>false</enableCaptcha>
</securityRealm>
```
删除上述部分，重启服务，发现可以直接进入系统了。

然后去系统管理里面，重新添加账号就可以了。

有个问题没有搞懂：
>之前创建的账号test,后面还可以创建一个同名账号test ,最终系统里面只显示一个test，不明白原因。可能是第一次账号创建没有成功把。
