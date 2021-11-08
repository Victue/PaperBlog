---
title: Dependency-Track中文文档
author: Li Pie
categories: Documents
tags: Dependency-Track
date: 2021-07-12
---

持续更新中，原文档：https://docs.dependencytrack.org/

------

#### 目录

> 介绍
>
> 开始
>
> 部署
>
> Docker
>
> Executable WAR
>
> WAR
>
> 初始化
>
> 配置
>
> 数据库支持
>
> 数据目录
>
> LDAP配置
>
> OpenID连接配置
>
> 使用
>
> 集成和交付
>
> 透明度
>
> 因素分析
>
> 策略遵从
>
> 采购
>
> 分析类型
>
> 已知漏洞分析
>
> 过时组件分析
>
> 许可证评估
>
> 组件标识
>
> 完整性验证
>
> 数据源
>
> NVD
>
> NPM Public Advisories
>
> Sonatype OSS Index
>
> VulnDB
>
> Datasource Routing
>
> 库文件
>
> 内部组件
>
> 私有漏洞库
>
> 报告结果
>
> 审计
>
> 状态
>
> 抑制
>
> 集成

## 初始化

初始化时会进行以下操作：

- 生成默认对象，如用户、团队、权限

- 生成用于JWT令牌创建和验证的密钥

- 加载CWE和SPDX license数据

- 加载漏洞数据源的镜像（NVD、NPM Advisories等）

**默认管理员账户**

- username:admin

- password:admin

第一次登录，需要更改密码。



## 配置

### 后端



默认情况下，配置文件`application.properties`在WAR的classpath中。可配置众多性能调整参数，最关键是外部数据源、目录服务(LDAP)和代理设置。

对于容器化部署，配置文件中定义的属性也可以指定为环境变量。所有环境变量都是大写的，句点（.）替换为下划线（_）。有关使用环境变量的配置示例，请参阅[Docker说明](https://docs.dependencytrack.org/getting-started/deploy-docker/)。

> 默认的嵌入式H2数据库设计用于快速评估和测试。不要在生产环境中使用嵌入式H2数据库。
>
>  请参阅：[数据库支持](https://docs.dependencytrack.org/getting-started/database-support/)。

如果需要使用自定义配置启动，请在执行时添加系统属性`alpine.application.properties`，例如：

```
-Dalpine.application.properties=~/.dependency-track/application.properties
```

#### 默认配置

```properties
############################ Alpine Configuration ###########################

# Required
# 设定the event subsystem使用的线程数量
# 由the event subsystem异步执行
# 当该值较大时，可以处理大多数使用场景并且不会产生大量延迟.
# 当该值较小时，不会对服务器造成额外的负载
# 值为0时，将指示Alpine为每个CPU核心分配1个线程
# 可以使用alpine.worker.thread.multiplier属性进行进一步调整.
# 默认为0
alpine.worker.threads=0

# Required
# 设置一个乘数，用于计算the event subsystem使用的线程数
# 此属性仅在alpine.worker.threads设置为0时使用
# 具有4个内核且乘数为4的计算机将使用（最多）16个工作线程
# 默认为4
alpine.worker.thread.multiplier=4

# Required
# 设定数据目录的路径。
# 此目录将保存日志、密钥和任何数据库或索引文件以及特定于应用程序的文件或目录
alpine.data.directory=~/.dependency-track

# Required
# 日志记录间隔时间(秒)
# 值为0时，不启用日志记录
alpine.watchdog.logging.interval=0

# Required
# 设定数据库操作模式。有效的选择是:
# 'server'， 'embedded'和'external'。
# server模式下，数据库将监听来自远程主机的连接。
# embedded模式下，系统会更安全，速度会较快。
# external模式下，使用外部数据库服务器
# (例如mysql, postgresql等)。
alpine.database.mode=embedded

# Optional
# 'server'模式下，指定TCP端口
alpine.database.port=9092

# Required
# 指定连接到数据库时要使用的JDBC URL
alpine.database.url=jdbc:h2:~/.dependency-track/db

# Required
# 指定要使用的JDBC driver class
alpine.database.driver=org.h2.Driver

# Optional
# 指定对数据库进行身份验证时要使用的用户名
alpine.database.username=sa

# Optional
# 指定对数据库进行身份验证时要使用的密码
# alpine.database.password=

# Optional
# 是否启用数据库连接池
alpine.database.pool.enabled=true

# Optional
# 设定数据库连接池最大连接数
# 包括空闲和正在使用的连接数
alpine.database.pool.max.size=20

# Optional
# 可在连接池中的最长空闲时间
alpine.database.pool.idle.timeout=600000

# Optional
# 连接池中连接的最长生命周期
# 正在使用的连接将不会被删除，只有关闭的连接才会被移除
alpine.database.pool.max.lifetime=600000

# Optional
# 设定强制验证，需要API密钥来实现自动化
# 用户界面将通过提示输入登录凭据来防止匿名访问
alpine.enforce.authentication=true

# Optional
# 设定强制授权，API密钥和用户帐户被限制为团队有权访问
# 若强制授权，alpine.enforce.authentication必须为true
alpine.enforce.authorization=true

# Required
# 设定对用户密码进行hash时要进行的bcrypt轮数
# 轮数越多，密码越安全，但会花费更多时间和资源
alpine.bcrypt.rounds=14

# Required
# 设定LDAP是否用于身份验证
# 如果为true，alpine.ldap.* 也应该进行相应设置
alpine.ldap.enabled=false

# Optional
# 设定LDAP服务URL
# Example (Microsoft Active Directory):
#    alpine.ldap.server.url=ldap://ldap.example.com:3268
#    alpine.ldap.server.url=ldaps://ldap.example.com:3269
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.server.url=ldap://ldap.example.com:389
#    alpine.ldap.server.url=ldaps://ldap.example.com:636
alpine.ldap.server.url=ldap://ldap.example.com:389

# Optional
# 设定base DN，所有查询都从中搜索
alpine.ldap.basedn=dc=example,dc=com

# Optional
# 设定LDAP安全验证级别，"none", "simple", "strong" 
# 如果为空或未指定，则由服务提供者指定
alpine.ldap.security.auth=simple

# Optional
# 如果不允许匿名访问，请设置对目录具有有限访问权限的用户名，仅够执行搜索
# This should be the fully qualified DN of the user.
alpine.ldap.bind.username=

# Optional
# 设置密码
alpine.ldap.bind.password=

# Optional
# Specifies if the username entered during login needs to be formatted prior
# to asserting credentials against the directory. 
# 对于Active Directory, userPrincipal通常以domain结尾，
# 而samAccountName和其他目录服务器则并非如此。
# %s变量将替换为登录时声明的用户名
# Example (Microsoft Active Directory):
#    alpine.ldap.auth.username.format=%s@example.com
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.auth.username.format=%s
alpine.ldap.auth.username.format=%s@example.com

# Optional
# 设定标识用户ID的属性
# Example (Microsoft Active Directory):
#    alpine.ldap.attribute.name=userPrincipalName
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.attribute.name=uid
alpine.ldap.attribute.name=userPrincipalName

# Optional
# 用于存储用户电子邮件地址的LDAP属性
alpine.ldap.attribute.mail=mail

# Optional
# 设定用于从目录检索所有组的LDAP搜索过滤器。
# Example (Microsoft Active Directory):
#    alpine.ldap.groups.filter=(&(objectClass=group)(objectCategory=Group))
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.groups.filter=(&(objectClass=groupOfUniqueNames))
alpine.ldap.groups.filter=(&(objectClass=group)(objectCategory=Group))

# Optional
# 设定用于查询用户和检索用户所属组列表的LDAP搜索过滤器
# {USER_DN}变量将在运行时替换为users DN的实际值
# Example (Microsoft Active Directory):
#    alpine.ldap.user.groups.filter=(&(objectClass=group)(objectCategory=Group)(member={USER_DN}))
# Example (Microsoft Active Directory - with nested group support):
#    alpine.ldap.user.groups.filter=(member:1.2.840.113556.1.4.1941:={USER_DN})
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.user.groups.filter=(&(objectClass=groupOfUniqueNames)(uniqueMember={USER_DN}))
alpine.ldap.user.groups.filter=(member:1.2.840.113556.1.4.1941:={USER_DN})

# Optional
# Specifies the LDAP search filter used to search for groups by their name.
# The {SEARCH_TERM} variable will be substituted at runtime.
# Example (Microsoft Active Directory):
#    alpine.ldap.groups.search.filter=(&(objectClass=group)(objectCategory=Group)(cn=*{SEARCH_TERM}*))
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.groups.search.filter=(&(objectClass=groupOfUniqueNames)(cn=*{SEARCH_TERM}*))
alpine.ldap.groups.search.filter=(&(objectClass=group)(objectCategory=Group)(cn=*{SEARCH_TERM}*))

# Optional
# Specifies the LDAP search filter used to search for users by their name.
# The {SEARCH_TERM} variable will be substituted at runtime.
# Example (Microsoft Active Directory):
#    alpine.ldap.users.search.filter=(&(objectClass=group)(objectCategory=Group)(cn=*{SEARCH_TERM}*))
# Example (ApacheDS, Fedora 389 Directory, NetIQ/Novell eDirectory, etc):
#    alpine.ldap.users.search.filter=(&(objectClass=inetOrgPerson)(cn=*{SEARCH_TERM}*))
alpine.ldap.users.search.filter=(&(objectClass=user)(objectCategory=Person)(cn=*{SEARCH_TERM}*))

# Optional
# Specifies if mapped LDAP accounts are automatically created upon successful
# authentication. When a user logs in with valid credentials but an account has
# not been previously provisioned, an authentication failure will be returned.
# This allows admins to control specifically which ldap users can access the
# system and which users cannot. When this value is set to true, a local ldap
# user will be created and mapped to the ldap account automatically. This
# automatic provisioning only affects authentication, not authorization.
alpine.ldap.user.provisioning=false

# Optional
# This option will ensure that team memberships for LDAP users are dynamic and
# synchronized with membership of LDAP groups. When a team is mapped to an LDAP
# group, all local LDAP users will automatically be assigned to the team if
# they are a member of the group the team is mapped to. If the user is later
# removed from the LDAP group, they will also be removed from the team. This
# option provides the ability to dynamically control user permissions via an
# external directory.
alpine.ldap.team.synchronization=false

# Optional
# HTTP proxy
# 如果设置了address，那么port也必须设置。
# alpine.http.proxy.address=proxy.example.com
# alpine.http.proxy.port=8888
# alpine.http.proxy.username=
# alpine.http.proxy.password=
# alpine.no.proxy=localhost,127.0.0.1

# Optional
# 包含在REST响应中的跨源资源共享(CORS)头。
# 如果'alpine.cors.enabled' 为true, 将发送CORS头，false则不发送
# 请参阅: https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
# 以下为默认值
#alpine.cors.enabled=true
#alpine.cors.allow.origin=*
#alpine.cors.allow.methods=GET, POST, PUT, DELETE, OPTIONS
#alpine.cors.allow.headers=Origin, Content-Type, Authorization, X-Requested-With, Content-Length, Accept, Origin, X-Api-Key, X-Total-Count, *
#alpine.cors.expose.headers=Origin, Content-Type, Authorization, X-Requested-With, Content-Length, Accept, Origin, X-Api-Key, X-Total-Count
#alpine.cors.allow.credentials=true
#alpine.cors.max.age=3600

# Required
# 是否将OpenID Connect用于用户身份验证
# 如果启用，则应相应地设置alpine.oidc.*
alpine.oidc.enabled=false

# Optional
# Defines the issuer URL to be used for OpenID Connect.
# This issuer MUST support provider configuration via the /.well-known/openid-configuration endpoint.
# See also:
# - https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderMetadata
# - https://openid.net/specs/openid-connect-discovery-1_0.html#ProviderConfig
alpine.oidc.issuer=

# Optional
# Defines the name of the claim that contains the username in the provider's userinfo endpoint.
# Common claims are "name", "username", "preferred_username" or "nickname".
# See also: https://openid.net/specs/openid-connect-core-1_0.html#UserInfoResponse
alpine.oidc.username.claim=name

# Optional
# Specifies if mapped OpenID Connect accounts are automatically created upon successful
# authentication. When a user logs in with a valid access token but an account has
# not been previously provisioned, an authentication failure will be returned.
# This allows admins to control specifically which OpenID Connect users can access the
# system and which users cannot. When this value is set to true, a local OpenID Connect
# user will be created and mapped to the OpenID Connect account automatically. This
# automatic provisioning only affects authentication, not authorization.
alpine.oidc.user.provisioning=false

# Optional
# This option will ensure that team memberships for OpenID Connect users are dynamic and
# synchronized with membership of OpenID Connect groups or assigned roles. When a team is
# mapped to an OpenID Connect group, all local OpenID Connect users will automatically be
# assigned to the team if they are a member of the group the team is mapped to. If the user
# is later removed from the OpenID Connect group, they will also be removed from the team. This
# option provides the ability to dynamically control user permissions via the identity provider.
# Note that team synchronization is only performed during user provisioning and after successful
# authentication.
alpine.oidc.team.synchronization=false

# Optional
# Defines the name of the claim that contains group memberships or role assignments in the provider's userinfo endpoint.
# The claim must be an array of strings. Most public identity providers do not support group or role management.
# When using a customizable / on-demand hosted identity provider, name, content, and inclusion in the userinfo endpoint
# will most likely need to be configured.
alpine.oidc.teams.claim=groups
```

#### 代理配置

代理支持以下两种方式进行配置，使用`application.properties`中定义的代理设置或通过环境变量。默认情况下，系统将尝试读取 `https_proxy`，`http_proxy` 和 `no_proxy`环境变量。如果他们被设置，Dependency-Track将自动使用。

 `no_proxy`指定的URLs将会从代理中被排除。可以是以逗号分隔的主机名、域名或两者的混合列表。如果为某个URL指定了端口号，则只有具有该端口号的请求才会被排除在代理之外。不能将no_proxy设置为单个星号(“*”)以匹配所有主机。

支持需要BASIC、DIGEST和NTLM验证的代理。

#### 日志记录等级

可以通过在启动时设置`dependencyTrack.Logging.level`系统属性来指定日志记录等级(INFO、WARN、ERROR、DEBUG、TRACE)。例如，以下命令将启动带有调试日志记录的Dependency-Track (embedded) ：

```
java -Xmx4G -DdependencyTrack.logging.level=DEBUG -jar dependency-track-embedded.war
```

对于Docker部署，只需将`LOGGING_LEVEL`环境变量设置为INFO、WARN、ERROR、DEBUG或TRACE。



### 前端

前端使用`config.json`文件，该文件通过 AJAX 动态请求和评估。文件位于 `<BASE_URL>/static/config.json`。

**默认配置**

```json
{
    // Required
    // URL of the Dependency-Track backend
    "API_BASE_URL": "",
    // Optional
    // Defines the issuer URL to be used for OpenID Connect.
    // See alpine.oidc.issuer property of the backend.
    "OIDC_ISSUER": "",
    // Optional
    // Defines the client ID for OpenID Connect.
    "OIDC_CLIENT_ID": "",
    // Optional
    // Defines the scopes to request for OpenID Connect.
    // See also: https://openid.net/specs/openid-connect-basic-1_0.html#Scopes
    "OIDC_SCOPE": "openid profile email",
    // Optional
    // Specifies the OpenID Connect flow to use.
    // Values other than "implicit" will result in the Code+PKCE flow to be used.
    // Usage of the implicit flow is strongly discouraged, but may be necessary when
    // the IdP of choice does not support the Code+PKCE flow.
    // See also:
    //   - https://oauth.net/2/grant-types/implicit/
    //   - https://oauth.net/2/pkce/
    "OIDC_FLOW": "",
}
```

对于容器化部署，这些设置可以通过以下任一方式覆盖:

- 将自定义的`config.json`挂载到容器中`/app/static/config.json`
- 将其作为环境变量提供

环境变量的名称与`config.json`中的对应名称相同。

挂载的`config.json`优先于环境变量。如果两者都被设定，则环境变量将被忽略。



## 数据库支持

默认包含嵌入式数据库H2。主要用于快速评估、测试和描述Dependency-Track平台的功能。

> 请不要在实际工作中使用默认数据库。

支持以下数据库：

- Microsoft SQL Server 2012 and higher
- MySQL 5.6 and 5.7
- PostgreSQL 9.0 and higher

数据库设置配置文件为`application.properties`，位于数据目录(data directory)中。

#### Microsoft SQL Server Example

```properties
alpine.database.mode=external
alpine.database.url=jdbc:sqlserver://localhost:1433;databaseName=dtrack
alpine.database.driver=com.microsoft.sqlserver.jdbc.SQLServerDriver
alpine.database.username=dtrack
alpine.database.password=password
```

#### MySQL Example

```properties
alpine.database.mode=external
alpine.database.url=jdbc:mysql://localhost:3306/dtrack?autoReconnect=true&useSSL=false
alpine.database.driver=com.mysql.jdbc.Driver
alpine.database.username=dtrack
alpine.database.password=password
```

使用MySQL时，必须在创建数据库之前，从sql-mode中删除‘NO_ZERO_IN_DATE’ and ‘NO_ZERO_DATE’，以及添加 ‘ANSI_QUOTES’ 。具体请参照MySQL文档。

有多种方式改变MySQL配置文件，推荐使用以下方式(修改my.ini)：

```properties
[mysqld] 
sql_mode="ANSI_QUOTES,STRICT_TRANS_TABLES,ONLY_FULL_GROUP_BY,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION"
```

值得注意的是，MySQL会错误报告(“Specified key was too long”)，实际上并没有。如果需要使用UTF-8，请不要使用MySQL。

#### PostgreSQL Example

```properties
alpine.database.mode=external
alpine.database.url=jdbc:postgresql://localhost:5432/dtrack
alpine.database.driver=org.postgresql.Driver
alpine.database.username=dtrack
alpine.database.password=password
```



## 数据目录

在UNIX/Linux上使用` ~/.dependency-track`，在Windows上使用当前用户主目录的 `.dependency-track` 。该目录被称为数据目录(*data directory*)，包括NIST NVD镜像，嵌入式数据库文件，审计日志以及部分密钥。应尽量提升数据目录安全性。

| 数据                       | 用途                                    |
| -------------------------- | --------------------------------------- |
| db.mv.db                   | Embedded H2 database                    |
| dependency-track.log       | Application log                         |
| dependency-track-audit.log | Application audit log                   |
| id.system                  | Randomly generated system identifier    |
| index                      | Internal search engine index            |
| keys                       | Keys used to generate/verify JWT tokens |
| nist                       | Mirror of the NVD and CPE               |
| server.log                 | Embedded Jetty server log               |
| vulndb                     | Mirror of VulnDB                        |



## LDAP配置

Dependency-Track已在多种LDAP服务器上测试。下述内容为部分已测试可用于服务器默认模式的示例配置。

#### Microsoft Active Directory Example

```
alpine.ldap.enabled=true
alpine.ldap.server.url=ldap://ldap.example.com:3268
alpine.ldap.basedn=dc=example,dc=com
alpine.ldap.security.auth=simple
alpine.ldap.auth.username.format=%s@example.com
alpine.ldap.bind.username=cn=ServiceAccount,ou=Users,dc=example,dc=com
alpine.ldap.bind.password=mypassword
alpine.ldap.attribute.name=userPrincipalName
alpine.ldap.attribute.mail=mail
alpine.ldap.groups.filter=(&(objectClass=group)(objectCategory=Group))
alpine.ldap.user.groups.filter=(member:1.2.840.113556.1.4.1941:={USER_DN})
alpine.ldap.groups.search.filter=(&(objectClass=group)(objectCategory=Group)(cn=*{SEARCH_TERM}*))
alpine.ldap.users.search.filter=(&(objectClass=user)(objectCategory=Person)(cn=*{SEARCH_TERM}*))
```

#### ApacheDS Example

```
alpine.ldap.enabled=true
alpine.ldap.server.url=ldap://ldap.example.com:389
alpine.ldap.basedn=dc=example,dc=com
alpine.ldap.security.auth=simple
alpine.ldap.auth.username.format=%s
alpine.ldap.bind.username=uid=ServiceAccount,ou=system
alpine.ldap.bind.password=mypassword
alpine.ldap.attribute.name=cn
alpine.ldap.attribute.mail=mail
alpine.ldap.groups.filter=(&(objectClass=groupOfUniqueNames))
alpine.ldap.user.groups.filter=(&(objectClass=groupOfUniqueNames)(uniqueMember={USER_DN}))
alpine.ldap.groups.search.filter=(&(objectClass=groupOfUniqueNames)(cn=*{SEARCH_TERM}*))
alpine.ldap.users.search.filter=(&(objectClass=inetOrgPerson)(cn=*{SEARCH_TERM}*))
```

#### Fedora 389 Directory Example

```
alpine.ldap.enabled=true
alpine.ldap.server.url=ldap://ldap.example.com:389
alpine.ldap.basedn=dc=example,dc=com
alpine.ldap.security.auth=simple
alpine.ldap.auth.username.format=%s
alpine.ldap.bind.username=cn=directory manager
alpine.ldap.bind.password=mypassword
alpine.ldap.attribute.name=uid
alpine.ldap.attribute.mail=mail
alpine.ldap.groups.filter=(&(objectClass=groupOfUniqueNames))
alpine.ldap.user.groups.filter=(&(objectClass=groupOfUniqueNames)(uniqueMember={USER_DN}))
alpine.ldap.groups.search.filter=(&(objectClass=groupOfUniqueNames)(cn=*{SEARCH_TERM}*))
alpine.ldap.users.search.filter=(&(objectClass=inetOrgPerson)(cn=*{SEARCH_TERM}*))
```

#### NetIQ/Novell eDirectory Example

```
alpine.ldap.enabled=true
alpine.ldap.server.url=ldaps://ldap.example.com:636
alpine.ldap.basedn=o=example
alpine.ldap.security.auth=simple
alpine.ldap.auth.username.format=%s
alpine.ldap.bind.username=cn=ServiceAccount,o=example
alpine.ldap.bind.password=mypassword
alpine.ldap.attribute.name=uid
alpine.ldap.attribute.mail=mail
alpine.ldap.groups.filter=(&(objectClass=groupOfUniqueNames))
alpine.ldap.user.groups.filter=(&(objectClass=groupOfUniqueNames)(uniqueMember={USER_DN}))
alpine.ldap.groups.search.filter=(&(objectClass=groupOfUniqueNames)(cn=*{SEARCH_TERM}*))
alpine.ldap.users.search.filter=(&(objectClass=inetOrgPerson)(cn=*{SEARCH_TERM}*))
```



## OpenID Connect配置

```
仅支持Dependency-Track 4.0.0或更高版本。
```

在OAuth2 / OIDC环境中，Dependency-Track的前端充当客户机(*client*)，而API服务器充当资源服务器(*resource server*)(具体请参阅[OAuth2 roles](https://tools.ietf.org/html/rfc6749#section-1.1))。因此，前端需要额外的配置，目前只有在与API服务器分开部署时才支持这种配置。有关说明，请参阅[配置](https://docs.dependencytrack.org/getting-started/configuration/)和[Docker部署](https://docs.dependencytrack.org/getting-started/deploy-docker/)页面。典型的Dependency-Track不支持仅使用[WAR](https://docs.dependencytrack.org/getting-started/deploy-war/)或[excutable WAR](https://docs.dependencytrack.org/getting-started/deploy-exewar/)部署！

如果配置正确，用户将可以通过*OpenID* 登录：

![Login page with OpenID button](https://docs.dependencytrack.org/images/screenshots/oidc-login-page.png)

---

### 配置示例

Generally, Dependency-Track can be used with any identity provider that implements the [OpenID Connect](https://openid.net/connect/) standard.下面是一些已知有效的配置示例。请注意，某些providers可能不支持某些特性，例如团队同步，或者需要修改配置文件。如果发现选择的provider无法与Dependency-Track一起使用，请提交[issue](https://github.com/DependencyTrack/dependency-track/issues)。

有关前端和后端可用的完整配置设置，请参阅[配置页面](https://docs.dependencytrack.org/getting-started/configuration/)。

#### Auth0

| API server                                   | Frontend                                        |
| :------------------------------------------- | :---------------------------------------------- |
| alpine.oidc.enabled=true                     | OIDC_CLIENT_ID=9XgMg7bP7QbD74TZnzZ9Jhk9KHq3RPCM |
| alpine.oidc.issuer=https://example.auth0.com | OIDC_ISSUER=https://example.auth0.com           |
| alpine.oidc.username.claim=nickname          |                                                 |
| alpine.oidc.user.provisioning=true           |                                                 |
| alpine.oidc.teams.claim=groups*              |                                                 |
| alpine.oidc.team.synchronization=true*       |                                                 |

*需要[额外配置](https://auth0.com/docs/extensions/authorization-extension/use-rules-with-the-authorization-extension)

#### GitLab (gitlab.com)

| API server                            | Frontend                                                     |
| :------------------------------------ | :----------------------------------------------------------- |
| alpine.oidc.enabled=true              | OIDC_CLIENT_ID=ff53529a3806431e06b2930c07ab0275a9024a59873a0d5106dd67c4cd34e3be |
| alpine.oidc.issuer=https://gitlab.com | OIDC_ISSUER=https://gitlab.com                               |
| alpine.oidc.username.claim=nickname   |                                                              |
| alpine.oidc.user.provisioning=true    |                                                              |
| alpine.oidc.teams.claim=groups        |                                                              |
| alpine.oidc.team.synchronization=true |                                                              |

> gitlab.com目前没有所需要设置的CORS headers，具体请参阅GitLab issue [#209259.](https://gitlab.com/gitlab-org/gitlab/-/issues/209259)
>
> 对于内部安装，可以通过反向代理设置所需的headers来解决这个问题。

#### Keycloak

| API server                                                   | Frontend                                                 |
| :----------------------------------------------------------- | :------------------------------------------------------- |
| alpine.oidc.enabled=true                                     | OIDC_CLIENT_ID=dependency-track                          |
| alpine.oidc.issuer=https://auth.example.com/auth/realms/example | OIDC_ISSUER=https://auth.example.com/auth/realms/example |
| alpine.oidc.username.claim=preferred_username                |                                                          |
| alpine.oidc.user.provisioning=true                           |                                                          |
| alpine.oidc.teams.claim=groups*                              |                                                          |
| alpine.oidc.team.synchronization=true*                       |                                                          |

*需要额外配置，参阅[Keycloak示例](https://docs.dependencytrack.org/getting-started/openidconnect-configuration/#example-setup-with-keycloak)

---

### Keycloak示例

以下步骤演示如何设置将OpenID Connect与Keycloak配合使用。大多数设置可以应用在其他idPs上。

> This guide assumes that:
>
> - the Dependency-Track frontend has been deployed to `https://dependencytrack.example.com`
> - a Keycloak instance is available at `https://auth.example.com`
> - the realm *example* has been created in Keycloak

1. 如下所示配置客户端：

   ![Keycloak: Configure client](https://docs.dependencytrack.org/images/screenshots/oidc-keycloak-client-settings.png)

   - Client ID: `dependency-track`
   - Client Protocol: `openid-connect`
   - Access Type: `public`
   - Standard Flow Enabled: `ON`
   - Valid Redirect URIs: `https://dependencytrack.example.com/static/oidc-callback.html*`
   - The trailing `*` is required when using the frontend v1.3.0 or newer, in order to support [post-login redirects](https://github.com/DependencyTrack/frontend/pull/47)
   - Web Origins: `https://dependencytrack.example.com`

2. 为了能够同步团队成员身份，需要创建一个*protocol mapper*，将组成员身份作为 `groups` 包含在 `/userinfo` 端节点中：

   ![Keycloak: Create protocol mapper for groups](https://docs.dependencytrack.org/images/screenshots/oidc-keycloak-create-protocol-mapper.png)

   - Mapper Type: `Group Membership`
   - Token Claim Name: `groups`
   - Add to userinfo: `ON`



# 使用

## 集成和交付

Dependency Track高速处理和分析CycloneDX BOMs，非常适合在modern build pipelines中使用。CycloneDX  BOMs的生成通常发生在CI期间或生成最终应用程序集时。由于构建时工具的可用性，CycloneDX是首选的BOM格式，这些格式侧重于安全用例。但是，也支持SPDX tag和RDF格式的BOMs。

访问[CycloneDX Tool Center](https://cyclonedx.org/tool-center/)，了解生成CycloneDX BOMs可用工具的信息。

> Dependency-Track持续监视组件是否存在已知漏洞。在Dependency-Track中添加或更新组件时，将对该组件执行分析。此操作发生在通过REST或从用户界面接收文件以及更改组件的过程中。Dependency-Track将会保持每天自动分析组件安全。

在Jenkins环境中，推荐使用[Dependency Track Jenkins Plugin](https://plugins.jenkins.io/dependency-track/)将CycloneDX BOMs上传至Dependency-Track。

对于其他环境，可以使用cURL(或类似的)。

#### CycloneDX or SPDX BOM

要发布CycloneDX或SPDX BOMs，请使用有效的API key和项目UUID。Base64对bom进行编码，并将结果文本插入“bom”字段。

```
curl -X "PUT" "http://dtrack.example.com/api/v1/bom" \
     -H 'Content-Type: application/json' \
     -H 'X-API-Key: LPojpCDSsEd4V9Zi6qCWr4KsiF3Konze' \
     -d $'{
  "project": "f90934f5-cb88-47ce-81cb-db06fc67d4b4",
  "bom": "PD94bWwgdm..."
  }'
```

也可以通过HTTP POST发布BOMs，不需要Base64编码。

```
curl -X "POST" "http://dtrack.example.com/api/v1/bom" \
     -H 'Content-Type: multipart/form-data; charset=utf-8; boundary=__X_CURL_BOUNDARY__' \
     -H 'X-Api-Key: LPojpCDSsEd4V9Zi6qCWr4KsiF3Konze' \
     -F "project=f90934f5-cb88-47ce-81cb-db06fc67d4b4" \
     -F "bom=<?xml version=\"1.0\" encoding=\"UTF-8\"?>..."
```

#### 大型负载

在上传的扫描或BOM较大的情况下，可以优选使用cURLs功能来指定包含有效负载的文件。

```
curl -X "PUT" "http://dtrack.example.com/api/v1/bom" \
     -H 'Content-Type: application/json' \
     -H 'X-API-Key: LPojpCDSsEd4V9Zi6qCWr4KsiF3Konze' \
     -d @payload.json
```



## 保持信息透明

Dependency-Track主要关注在SBOMs的消耗和分析上。然而，Dependency-Track也能够从项目集合(portfolio)中的任何项目生成SBOMs。组织能够在客户、合作伙伴或其他利益相关者请求时为项目集合(portfolio)中的任何项目创建SBOMs。

需要更高透明度的组织可以选择使用Dependency-Track通知功能，该功能能够在系统使用或处理SBOM时通过webhook发布SBOMs。当在持续集成或交付环境中使用时，SBOMs可以选择性地发布到一个或多个端点，从而实现与预定方的信息透明。

虽然保持透明是可能的，但应仔细考虑透明度。鼓励各组织首先与同一组织内的其他部门或业务单位共享SBOM数据，然后再与外部各方共享数据。

请参阅[通知](https://docs.dependencytrack.org/integrations/notifications/)，以获取有关通过webhooks在 `BOM_CONSUMED`和 `BOM_PROCESSED` 上共享的SBOM数据信息。



## 影响分析



## 策略制定



# 分析类型

## 已知漏洞分析

Dependency-Track集成多个漏洞数据源，识别具有已知漏洞的组件。该平台采用多种漏洞识别方法，包括：

| 分析器    | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| Internal  | Identifies vulnerable components from an internal directory of vulnerable software |
| NPM Audit | NPM Audit is a service which identifies vulnerabilities in Node.js Modules |
| OSS Index | OSS Index is a service provided by Sonatype which identifies vulnerabilities in third-party components |
| VulnDB    | VulnDB is a commercial service which identifies vulnerabilities in third-party components |

上述每种分析器都可以彼此独立地启用或禁用。

---

### Internal Analyzer

内部分析依赖于易受攻击软件的字典。当执行NVD镜像或VulnDB镜像时，将自动填充此字典。内部分析仪适用于所有具有有效CPE的组件，包括应用程序、操作系统和硬件组件。

----

### NPM Audit Analyzer

NPM Audit是一种识别Node.js模块中漏洞的服务。依赖跟踪与NPM Audit service本地集成，以提供高度准确的结果。使用此分析器需要所分析组件的有效的Package URLs。有关更多信息，请参阅[NPM Public Advisories (Datasource)](https://docs.dependencytrack.org/datasources/npm/) 。

---

### OSS Index Analyzer

OSS Index是Sonatype提供的一项服务，用于识别第三方组件中的漏洞。该服务支持广泛的包管理生态系统。Dependency-Track默认与OSS Index集成，提供高度准确的结果。此分析器适用于具有有效Package URLs所有组件。

> 从Dependency Track  v4.0开始，OSS Index默认启用，不需要帐户。对于以前的版本，OSS Index在默认情况下是禁用的，并且需要一个帐户。要启用OSS索引，请注册一个免费帐户，并在Dependency-Track的Analyzers设置的administrative console中输入帐户详细信息。

有关OSS Index漏洞数据更多信息，请参阅[OSS Index (Datasource)](https://docs.dependencytrack.org/datasources/ossindex/)。

---

### VulnDB Analyzer

VulnDB是Risk Based Security提供的订阅服务。VulnDB分析器能够根据VulnDB服务分析所有CPEs组件。使用该分析器需要组件的CPE信息。

---

### Analysis Interval Throttle

Dependency-Track包含一个内部限制器，用于在执行漏洞分析时防止对远程服务的重复请求。当一个组件Package URL或CPE被成功地用于一个给定的分析器时，时间戳将被记录下来并与interval throttle 进行比较。interval throttle 默认为24小时。



## 过时组件分析

软件链中使用的具有已知漏洞的组件对依赖于这些组件的项目来说具有重大风险。但是，使用不存在已知漏洞但不是最新版本的组件也存在以下风险：

- 很大一部分漏洞是被发现和修复的，从未通过官方渠道报告过，或者在很晚的时候报告过。使用具有未报告（但仍然已知）漏洞的组件仍然会给依赖它们的项目带来风险。 
- 针对组件报告漏洞的可能性相当高。组件在发布之间可能有轻微的API更改，或者在主要发布之间有显著的API更改。保持组件更新是实现以下目的的最佳做法：
  - 从性能、稳定性和其他错误修复中获益
  - 受益于其他特性和功能
  - 受益于持续的社区支持
  - 发现漏洞时快速响应安全事件

通过不断更新组件，团队可以更好地在已知影响组件的漏洞时进行快速响应。



## 许可证评估

作为Dependency-Track策略引擎的一部分，可以根据一个或多个策略评估许可证。



## 组件标识

作为Dependency-Track策略引擎的一部分，标识包括：

| 标识        | 描述                             |
| ----------- | -------------------------------- |
| Coordinates | 匹配包含指定组、名称和版本的组件 |
| Package URL | 匹配指定Package URL的组件        |
| CPE         | 匹配指定CPE的组件                |
| SWID TagID  | 匹配指定SWID TagID的组件         |
| Hash        | 匹配指定Hash的组件               |

- Hash标识自动检测所有已支持的hash算法：
  - MD5
  - SHA-1
  - SHA-256
  - SHA-384
  - SHA-512
  - SHA3-256
  - SHA3-384
  - SHA3-512
  - BLAKE2b-256
  - BLAKE2b-384
  - BLAKE2b-512
  - BLAKE3

### 使用

基于标识评估组件的通常使用于：

- 策略包含预定义的允许或禁止的组件列表

- 识别伪造或已知的恶意组件



## 完整性验证

Work in progress



# 数据源

## NVD

The National Vulnerability Database(NVD)是最大的公开漏洞信息源。由美国国家标准与技术研究所(NIST)内的一个团队维护，并以MITRE和其他人的工作为基础。NVD中的漏洞称为Common Vulnerabilities and Exposures(CVE)。从20世纪90年代到现在，NVD记录了超过100000个CVEs。

Dependency-Track严重依赖于NVD提供的数据，并包含一个完整的镜像，该镜像每天保持最新，或者在Dependency-Track实例重新启动时加载最新数据。

Credit提供数据来源的部分信息，以及CVE链接。

---

### NVD镜像

Dependency-Track不仅是NVD的使用者，而且还提供NVD镜像功能。此功能内置于Dependency-Track中，不需要进一步配置，镜像每天自动更新。

> NVD镜像URL: http://hostname/mirror/nvd

目录列表被禁止提供，但是索引由NVD提供的内容组成。这包括：

**JSON 1.1 feed**

- nvdcve-1.1-modified.json.gz
- nvdcve-1.1-%d.json.gz
- nvdcve-1.1-%d.meta

（其中%d是从2002年开始的四位数年份）



## NPM Public Advisories

NPM public advisories是JavaScript和Node.js漏洞信息的集中来源，NVD中可能记录也可能没有记录。利用Node.js的项目将受益于NPM Audit数据源，因为它提供了对特定于生态漏洞的可见性。

Dependency-Track与NPM集成，使用其public advisory API。Dependency Track能够创建所有NPM advisory数据的镜像。镜像每天都保持最新，或者在重新启动Dependency-Track实例时加载最新数据。

Credit提供数据来源的部分信息，以及NPM  advisories链接。





## Sonatype OSS Index

Sonatype OSS Index为具有有效的Package URLs的组件提供透明且高度准确的结果。大多数漏洞直接映射到国家漏洞数据库(NVD)中的CVE，但是OSS Index中确实包含许多NVD中不存在的漏洞。

Dependency Track与OSS Index集成，使用其public API。Dependency-Track没有镜像OSS Index，但它会在“as-identified”的基础上识别OSS Index中的漏洞信息。

> 从Dependency Track  v4.0开始，OSS Index默认启用，不需要帐户。对于以前的版本，OSS Index在默认情况下是禁用的，并且需要一个帐户。要启用OSS Index，请注册一个免费帐户，并在administrative的“Analyzers”设置中输入帐户详细信息。



## VulnDB



## Datasource Routing



## Repositories



## Internal Components



## 私有漏洞库

> 此功能在Dependency-Track v3.x中是实验性的，v4.x中还不可用。

Dependency-Track能够维护自己的内部管理漏洞存储库。私有存储库的行为与其他漏洞信息源（如NVD）相同。

![add vulnerability](https://docs.dependencytrack.org/images/screenshots/vulnerability-add.png)

私有漏洞库有三个主要用例。

- 希望跟踪内部开发的组件中漏洞的组织，这些组件在组织中的各种软件项目之间共享。 
- 执行安全研究的组织需要在选择性地披露之前记录所述研究。 
- 使用非托管数据源来识别漏洞的组织。这包括： 
  - 更改日志
  - 提交日志
  - 问题跟踪程序
  - 社交媒体信息

私有漏洞存储库中跟踪漏洞的来源为“INTERNAL”。与系统中的其他漏洞一样，需要一个惟一的`VulnID`来帮助惟一地标识每个漏洞。建议组织遵循相应规则来帮助识别源。例如，NVD中的漏洞都以“CVE-”开头。同样，跟踪自身漏洞的组织可能会选择使用“ACME-”或“INT-”之类的标记，或者根据漏洞的类型使用多个限定符。唯一的要求是VulnID对于INTERNAL是唯一的。

# 报告结果



## 审计



## 状态



## 抑制



# 集成

## Ecosystem Overview

## File Formats

## Fortify ssc

## Code Dx

## Kenna Security

## DefectDojo

## Notifications

## REST API

Dependency-Track is built using a *thin server architecture* and an *API-first design*。API是平台的核心。每个API都通过Swagger 2.0完整记录。

> http://{hostname}:{port}/api/swagger.json

Swagger UI控制台(平台不包括)可用于可视化和探索各种可能性。Chrome和FireFox扩展可以用来快速使用Swagger UI控制台。

![Swagger UI Console](https://docs.dependencytrack.org/images/screenshots/swagger-ui-console.png)

在使用REST APIs之前，必须生成API密钥。默认情况下，创建一个team还将创建相应的API密钥。一个team可能有多个密钥。

![Teams - API Key](https://docs.dependencytrack.org/images/screenshots/teams.png)



## ThreadFix

## SVG Badges

## Community Integrations



# 最佳实践





# 常见问题





# 专业术语

### API Key

一串长的随机生成的数字，用于身份验证。所有REST APIs都使用API密钥进行身份验证。API密钥按团队分配。一个团队可能分配了0个或多个API Key。

---

### Auditing

评估调查结果的过程，以确定调查结果的准确性及其对组成部分和项目的影响。审计处理操作创建一个审计跟踪，获取跟踪每一个被发现的行为状态。

---

### Bill of Materials (BOM)

物料清单（BOM）定义并描述了项目软件中使用的内容。在软件供应链中，这是指与软件捆绑在一起的所有组件的内容，包括作者、发布者、名称、版本、许可证和版权。依赖项跟踪支持CycloneDX格式。特定的软件组件的物料清单通常称为SBOM。

---

### Component

将component组件定义为独立实体。组件可以是开源组件、第三方库、第一方库、操作系统或硬件设备。

---

### CPE

Common Platform Enumeration (CPE) 是信息技术系统、软件和软件包的结构化命名方案。基于统一资源标识符（URI）的通用语法，CPE通常包括供应商、产品名称和版本。

---

### CVE

