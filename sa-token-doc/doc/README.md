<p align="center">
	<img alt="logo" src="https://sa-token.dev33.cn/doc/logo.png" width="150" height="150">
</p>
<h1 align="center" style="margin: 30px 0 30px; font-weight: bold;">Sa-Token v1.30.0</h1>
<h5 align="center">一个轻量级 Java 权限认证框架，让鉴权变得简单、优雅！</h5>
<p align="center" class="badge-box">
	<a href="https://gitee.com/dromara/sa-token/stargazers"><img src="https://gitee.com/dromara/sa-token/badge/star.svg?theme=gvp"></a>
	<a href="https://gitee.com/dromara/sa-token/members"><img src="https://gitee.com/dromara/sa-token/badge/fork.svg?theme=gvp"></a>
	<a href="https://github.com/dromara/sa-token/stargazers"><img src="https://img.shields.io/github/stars/dromara/sa-token?style=flat-square&logo=GitHub"></a>
	<a href="https://github.com/dromara/sa-token/network/members"><img src="https://img.shields.io/github/forks/dromara/sa-token?style=flat-square&logo=GitHub"></a>
	<a href="https://github.com/dromara/sa-token/watchers"><img src="https://img.shields.io/github/watchers/dromara/sa-token?style=flat-square&logo=GitHub"></a>
	<a href="https://github.com/dromara/sa-token/issues"><img src="https://img.shields.io/github/issues/dromara/sa-token.svg?style=flat-square&logo=GitHub"></a>
	<a href="https://github.com/dromara/sa-token/blob/master/LICENSE"><img src="https://img.shields.io/github/license/dromara/sa-token.svg?style=flat-square"></a>
</p>

---

## 前言：️️
为了保证新同学不迷路，请允许我唠叨一下：无论您从何处看到本篇文章，最新开发文档永远在：[http://sa-token.dev33.cn/](http://sa-token.dev33.cn/)，
建议收藏在浏览器书签，如果您已经身处本网站下，则请忽略此条说明。

本文档将会尽力讲解每个功能的设计原因、应用场景，用心阅读文档，你学习到的将不止是 `Sa-Token` 框架本身，更是绝大多数场景下权限设计的最佳实践。


## Sa-Token 介绍

**Sa-Token** 是一个轻量级 Java 权限认证框架，主要解决：**登录认证**、**权限认证**、**单点登录**、**OAuth2.0**、**分布式Session会话**、**微服务网关鉴权**
等一系列权限相关问题。

Sa-Token 的 API 设计非常简单，有多简单呢？以登录认证为例，你只需要：

``` java
// 在登录时写入当前会话的账号id
StpUtil.login(10001);

// 然后在需要校验登录处调用以下方法：
// 如果当前会话未登录，这句代码会抛出 `NotLoginException` 异常
StpUtil.checkLogin();
```

至此，我们已经借助 Sa-Token 完成登录认证！

此时的你小脑袋可能飘满了问号，就这么简单？自定义 Realm 呢？全局过滤器呢？我不用写各种配置文件吗？

没错，在 Sa-Token 中，登录认证就是如此简单，不需要任何的复杂前置工作，只需这一行简单的API调用，就可以完成会话登录认证！

当你受够 Shiro、SpringSecurity 等框架的三拜九叩之后，你就会明白，相对于这些传统老牌框架，Sa-Token 的 API 设计是多么的简单、优雅！

权限认证示例（只有具备 `user:add` 权限的会话才可以进入请求）
``` java
@SaCheckPermission("user:add")
@RequestMapping("/user/insert")
public String insert(SysUser user) {
	// ... 
	return "用户增加";
}
```

将某个账号踢下线（待到对方再次访问系统时会抛出`NotLoginException`异常）
``` java
// 将账号id为 10001 的会话踢下线 
StpUtil.kickout(10001);
```

在 Sa-Token 中，绝大多数功能都可以 **一行代码** 完成：
``` java
StpUtil.login(10001);    // 标记当前会话登录的账号id
StpUtil.getLoginId();    // 获取当前会话登录的账号id
StpUtil.isLogin();    // 获取当前会话是否已经登录, 返回true或false
StpUtil.logout();    // 当前会话注销登录
StpUtil.kickout(10001);    // 将账号为10001的会话踢下线
StpUtil.hasRole("super-admin");    // 查询当前账号是否含有指定角色标识, 返回true或false
StpUtil.hasPermission("user:add");    // 查询当前账号是否含有指定权限, 返回true或false
StpUtil.getSession();    // 获取当前账号id的Session
StpUtil.getSessionByLoginId(10001);    // 获取账号id为10001的Session
StpUtil.getTokenValueByLoginId(10001);    // 获取账号id为10001的token令牌值
StpUtil.login(10001, "PC");    // 指定设备类型登录，常用于“同端互斥登录”
StpUtil.kickout(10001, "PC");    // 指定账号指定设备类型踢下线 (不同端不受影响)
StpUtil.openSafe(120);    // 在当前会话开启二级认证，有效期为120秒 
StpUtil.checkSafe();    // 校验当前会话是否处于二级认证有效期内，校验失败会抛出异常 
StpUtil.switchTo(10044);    // 将当前会话身份临时切换为其它账号 
```

即使不运行测试，相信您也能意会到绝大多数 API 的用法。



## Sa-Token 功能一览

- **登录认证** —— 单端登录、多端登录、同端互斥登录、七天内免登录
- **权限认证** —— 权限认证、角色认证、会话二级认证
- **Session会话** —— 全端共享Session、单端独享Session、自定义Session 
- **踢人下线** —— 根据账号id踢人下线、根据Token值踢人下线
- **账号封禁** —— 指定天数封禁、永久封禁、设定解封时间 
- **持久层扩展** —— 可集成Redis、Memcached等专业缓存中间件，重启数据不丢失
- **分布式会话** —— 提供jwt集成、共享数据中心两种分布式会话方案
- **微服务网关鉴权** —— 适配Gateway、ShenYu、Zuul等常见网关的路由拦截认证
- **单点登录** —— 内置三种单点登录模式：无论是否跨域、是否共享Redis，都可以搞定
- **OAuth2.0认证** —— 轻松搭建 OAuth2.0 服务，支持openid模式 
- **二级认证** —— 在已登录的基础上再次认证，保证安全性 
- **Basic认证** —— 一行代码接入 Http Basic 认证 
- **独立Redis** —— 将权限缓存与业务缓存分离 
- **临时Token验证** —— 解决短时间的Token授权问题
- **模拟他人账号** —— 实时操作任意用户状态数据
- **临时身份切换** —— 将会话身份临时切换为其它账号
- **前后台分离** —— APP、小程序等不支持Cookie的终端
- **同端互斥登录** —— 像QQ一样手机电脑同时在线，但是两个手机上互斥登录
- **多账号认证体系** —— 比如一个商城项目的user表和admin表分开鉴权
- **花式token生成** —— 内置六种Token风格，还可：自定义Token生成策略、自定义Token前缀
- **注解式鉴权** —— 优雅的将鉴权与业务代码分离
- **路由拦截式鉴权** —— 根据路由拦截鉴权，可适配restful模式
- **自动续签** —— 提供两种Token过期策略，灵活搭配使用，还可自动续签
- **会话治理** —— 提供方便灵活的会话查询接口
- **记住我模式** —— 适配[记住我]模式，重启浏览器免验证
- **密码加密** —— 提供密码加密模块，可快速MD5、SHA1、SHA256、AES、RSA加密 
- **全局侦听器** —— 在用户登陆、注销、被踢下线等关键性操作时进行一些AOP操作
- **开箱即用** —— 提供SpringMVC、WebFlux等常见web框架starter集成包，真正的开箱即用
- **更多功能正在集成中...** —— 如有您有好想法或者建议，欢迎加群交流


## 开源仓库 Star 趋势

<p class="un-dec-a-pre"></p>

[![github-chart](https://starchart.cc/dromara/sa-token.svg 'GitHub')](https://starchart.cc/dromara/sa-token)

如果 Sa-Token 帮助到了您，希望您可以为其点上一个 `star`：
[码云](https://gitee.com/dromara/sa-token)、
[GitHub](https://github.com/dromara/sa-token)


## 使用Sa-Token的开源项目 
参考：[Sa-Token 生态](/more/link)


## 交流群
QQ交流群：496757342 [点击加入](https://jq.qq.com/?_wv=1027&k=WNggbsFe)

微信交流群：

![微信群](https://dev33-test.oss-cn-beijing.aliyuncs.com/sa-token/i-wx-qr.png ':size=230')

（扫码添加微信，备注：sa-token，邀您加入群聊）

<br>
