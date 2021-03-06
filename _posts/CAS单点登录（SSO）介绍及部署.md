---
title: CAS单点登录（SSO）介绍及部署
date: 2017-11-9 23:19:23
tags:
- CAS单点登录
- 单点登录
- CAS
categories:
- 框架
- 中间件
---

# CAS介绍

* [CAS](https://apereo.github.io/cas) 是Yale（耶鲁）大学一个开源的企业级单点登录系统，它的特点：
  1. Java (Spring Webflow/Spring Boot) 服务组件
  1. 可插拔身份验证支持（LDAP，Database，X.509，MFA）
  1. 支持多种协议（CAS，SAML，OAuth，OpenID，OIDC）
  1. 跨平台客户端支持（Java，.Net，PHP，Perl，Apache等）
  1. 与uPortal，Liferay，BlueSocket，Moodle，Google Apps等集成

CAS提供了一个友好的开源社区，方便开发者积极支持和贡献项目。
<!-- more -->
# 知识扩展

## SSO

单点登录（Single Sign On），简称为 SSO，是目前比较流行的企业业务整合的解决方案之一。SSO的定义是在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。

* 业务流程
  1. 当用户第一次访问应用系统A的时候，因为还没有登录会被引导到<strong>认证系统</strong>中进行登录；根据用户提供的登录信息，认证系统进行身份校验，如果通过校验，应该返回给用户一个认证的凭据（ticket）；用户再访问别的应用的时候，就会将这个ticket带上，作为自己认证的凭据，应用系统接受到请求之后会把ticket送到认证系统进行校验，检查ticket的合法性。如果通过校验，用户就可以在不用再次登录的情况下访问应用系统B和应用系统C了。
* 认证系统功能要求
  1. 所有应用系统共享同一个身份认证系统
  1. 统一的认证系统是SSO的前提之一。认证系统的主要功能是将用户的登录信息和用户信息库相比较，对用户进行登录认证；认证成功后，认证系统应该生成统一的认证标志（ticket）并立即返还给用户。另外，认证系统还应该对ticket进行效验，判断其有效性。
  1. 所有应用系统能够识别和提取ticket信息
  1. 要实现SSO的功能，让用户只登录一次，就必须让应用系统能够识别已经登录过的用户。应用系统应该能对ticket进行识别和提取，通过与认证系统的通讯，能自动判断当前用户是否登录过，从而完成单点登录的功能。

## OAuth 2.0

OAuth是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。

* 业务流程
  1. OAuth在“客户端”与“服务提供商”之间设置了一个授权层（authorization layer）。“客户端”不能直接登录“服务提供商”，只能登录授权层，以此将用户与客户端区分开来。“客户端”登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。
  1. “客户端”登录授权层以后，“服务提供商”根据令牌的权限范围和有效期，向“客户端”开放用户储存的资料。

## 小结

通过上面的概述可以很清楚的想明白两者的适用场景，以面向的用户不同可以做个划分，sso服务对象是各个应用系统，oauth服务对象是用户。

# 入门

## 架构

![](/images/cas_architecture.png)

## 系统组件

CAS系统架构包括CAS Server和CAS Clients，它们是通过各种协议进行通信的两个物理组件。

### CAS Server

CAS Server是基于Spring框架构建的Java Servlet，其主要职责是通过发出和验证票证来验证用户并授予对使用CAS的服务（通常称为CAS客户端）的访问权限。当用户在服务器上登录时向会向用户发出票证（TGT）并创建SSO会话。通过使用TGT令牌进行浏览器重定向，根据用户的请求向服务发出服务票证（ST）。随后通过反向通道通信在CAS服务器上验证ST。这些业务流程在CAS协议文档中非常详细地描述。

### CAS Clients

“CAS Client”在具体使用中有两个不同的含义，CAS Client是可以通过所支持的协议与服务器通信的任何启用CAS的应用程序，CAS Client还可以是可以与各种软件平台和应用集成以便经由某种认证协议（例如，CAS，SAML，OAuth）与CAS Server通信的软件包。客户端支持多种软件平台和应用的CAS客户端已经开发。
* 平台
  1. Apache httpd Server (mod_auth_cas module)
  1. Java (Java CAS Client)
  1. NET (.NET CAS Client)
  1. PHP (phpCAS)
  1. Perl (PerlCAS)
  1. Python (pycas)
  1. Ruby (rubycas-client)
* 应用
  1. Outlook Web Application (ClearPass + .NET CAS Client)
  1. Atlassian Confluence
  1. Atlassian JIRA
  1. Drupal
  1. Liferay
  1. uPortal
* 支持的协议
  客户端通过几种支持的协议与服务器通信。所有支持的协议在概念上类似，但不同的协议对应于特定的应用场景。例如，CAS协议支持委托（代理）认证，SAML协议支持属性释放和单点退出。
  1. CAS (versions 1, 2, and 3)
  1. SAML 1.1 and 2
  1. OpenID Connect
  1. OpenID
  1. OAuth 2.0

# 部署

要想让CAS服务跑起来，具体的安装方式有很多，在这里只是演示我的方式，给大家一个参考。

* 安装要求
  1. java oracle jdk 1.8以上
  1. tomcat7以上
  1. maven3.x
  1. gradle2.x
* 下载安装包
  1. 你可以直接从github上下载这个项目 <code>https://github.com/apereo/cas-gradle-overlay-template</code>，也可以直接访问下载链接 <code>https://github.com/apereo/cas-gradle-overlay-template/archive/master.zip</code>。
  1. 有人可能好奇为什么要下载这个项目？原因在于我们要覆盖官方安装包中默认的属性，这也是官方提供的一个示例项目。
  1. 这个项目体积非常小，因为它依赖的jar包需要下载，这也就是为什么我要安装Maven的原因，但又由于这个项目包是由gradle来构建的，所以还需要安装Gradle。

## 终端操作

> 接下来大家跟着我的命令执行，应该会有相同的结果。

* 下载项目
```shell
cd /usr/local/src
wget -c https://github.com/apereo/cas-gradle-overlay-template/archive/master.zip
unzip master.zip

# 你应该看到和我相同的目录
├── cas-gradle-overlay-template
│   ├── LICENSE
│   ├── README.md
│   ├── build.gradle
│   ├── cas
│   ├── etc
│   ├── gradle
│   ├── gradle.properties
│   ├── gradlew
│   ├── gradlew.bat
│   └── settings.gradle
```

### 编译项目并下载依赖包

进入项目根目录，后续的命令除特别说明外都将在这个目录下执行。

```shell
cd
cas-gradle-overlay-template
```

在执行构建命令时会从远程地址下载 <code>gradle-3.3-bin.zip</code> 压缩包（下载速度慢，并且中断之后不会增量下载），所以我建议你通过旋风之类的软件把这个文件下载并放到下面的目录里面。

```shell
gradle
└── wrapper
    ├── gradle-3.3-bin.zip
    ├── gradle-wrapper.jar
    └── gradle-wrapper.properties
```

如果你像我上面说的那样做，那还别忘记要修改 <code>gradle-wrapper.properties</code> 文件，把原来的远程地址替换成相对地址即可。

```shell
vim gradle-wrapper.properties
distributionUrl=gradle-3.3-bin.zip
```

另外我建议你在maven中设置远程仓库地址为国内的源，因为这样下载速度会非常快，国外的仓库下载jar包太慢。国内的源里面推荐使用阿里的源，下面粘贴一段示例代码。

```shell
<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>
</mirror>
```

现在执行命令开始下载项目所需要的依赖包，并构建一个新的war包，我们会部署这个包以启动CAS服务。

```shell
./gradlew clean build
```

确保你在项目要目录下执行上面的命令，如果成功你会看到和我类似的结果

```
Copying configuration to /etc/cas/config
:cas:clean
:cas:compileJava UP-TO-DATE
:cas:processResources UP-TO-DATE
:cas:classes UP-TO-DATE
:cas:findMainClass
:cas:copyConfig
:cas:war
:cas:bootRepackage SKIPPED
:cas:assemble
:cas:compileTestJava UP-TO-DATE
:cas:processTestResources UP-TO-DATE
:cas:testClasses UP-TO-DATE
:cas:test UP-TO-DATE
:cas:check UP-TO-DATE
:cas:build

BUILD SUCCESSFUL
```

实际的结果可能和我不相同，因为我并非首次执行上面的命令，项目所依赖的包已经下载完了，所以你执行命令的时间可能比我要长。

上面的命令主要是为了让你下载项目的依赖jar包，虽然实际上我们的war包已经生成，但还没有配置一些参数以覆盖官方包中的默认参数，所以现在还得修改配置文件。具体介绍请继续往下看。

### 生成证书

由于CAS服务全程使用HTTPS加密通讯协议，所以必须配置证书以达到此目的，否则服务将不会正常服务。

不管你是从证书管理网站上申请的证书，还是像下面示例的这样，是自己生成的未签名证书，都可以按照下面的方法进行证书的配置。

第一步：生成key  `openssl genrsa -des3 -out cas.example.com.key`
第二步：根据上一步的key生成crt证书 `openssl req -new -x509 -key cas.example.com.key -out cas.example.com.crt`
第三步：根据前面两步的文件生成PKCS12证书 `openssl pkcs12 -export -in cas.example.com.crt -inkey cas.example.com.key -out cas.example.com.p12 -name cas
`
第四步：把P12证书导入java的密钥存储库文件中 `keytool -importkeystore -deststorepass 123456 -destkeypass 123456 -destkeystore keystore -srckeystore cas.example.com.p12 -srcstoretype PKCS12 -srcstorepass 123456 -alias cas
`

最后一步操作之后会在当前目录生成一个文件 <code>keystore</code>，你需要将这个文件复制到项目的配置文件目录下面。

```shell
mv keystore etc/cas/config/

-rw-r--r--  1 xxx  xxx   1.1K  2 18 17:17 etc/cas/config/keystore
```

### 参数配置

修改文件之后的内容示例

```shell
vim etc/cas/config/cas.properties

cas.server.name: https://cas.example.com:8443
cas.server.prefix: https://cas.example.com:8443/cas

cas.adminPagesSecurity.ip=127\.0\.0\.1

logging.config: file:/etc/cas/config/log4j2.xml

# Embedded Tomcat
server.ssl.keyStore: file:/etc/cas/config/keystore
server.ssl.keyStorePassword=123456
server.ssl.keyPassword=123456

# Accept Users Authentication
cas.authn.accept.users=

# Ticket Granting Cookie
cas.tgc.signingKey=Ci1kE5-PyQfD0i_a3sH16B32QhwGBbHXOmhR4r36vv0cB0RasLdEb7AI0ykouyMrE5RBbIAxqXvipmQEUA6juQ
cas.tgc.encryptionKey=Zv0LARlN7g7LxI6wmp6T4sLr2-TiZZ3K5W8pRIWcvO0

# Spring Webflow
cas.webflow.signing.key=16ua53AWy3PM4rYj6V0rBab_U-X7HvnFpDAVaXMEwwdhiZzTHM5vlYpLzm8HR6jf4DcDbM1_HQxCu6kQGjAOqg
cas.webflow.encryption.key=NamwVAyVrXeyrKvs
```

尽量按照上面的示例配置来修改，以免你掉进未知的“坑”里面。由于修改了配置文件，需要重新执行上面的命令以构建项目。

### 部署到容器

由于项目中使用了Spring boot，按照官方文档的描述是可以按照以下的命令进行启动的，但我在操作时每次都构建到80%左右终端上的命令就停止了。所以我选择直接将war包部署到tomcat中。

如果你使用Spring boot则使用下面的命令：`./gradlew bootRun`

如果你使用tomcat则按照如下步骤操作：

<ol>
<li>假设你安装了tomcat</li>
<li>把war包部署到webapps目录中 <code>cp cas/build/libs/cas.war /opt/tomcat/webapps/</code></li>
<li>启动tomcat <code>/opt/tomcat/bin/startup.sh</code></li>
<li>查询日志输出</li>
</ol>

```
18-Feb-2017 17:19:32.000 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
18-Feb-2017 17:19:32.026 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
18-Feb-2017 17:19:32.029 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["https-jsse-nio-8443"]
18-Feb-2017 17:19:32.411 信息 [main] org.apache.tomcat.util.net.NioSelectorPool.getSharedSelector Using a shared selector for servlet write/read
18-Feb-2017 17:19:32.412 信息 [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["ajp-nio-8009"]
```

## 结束

配置本地hosts让 <code>cas.example.com</code> 域名指向本地IP <code>127.0.0.1</code>，打开浏览器访问 <code>https://cas.example.com:8443/cas</code> 可以看到管理页面。
