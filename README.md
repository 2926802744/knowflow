# KnowFlow 后端

KnowFlow 是一个面向知识发布、内容互动与智能问答的社区项目。本仓库为后端服务，基于 Java 21、Spring Boot 3、MyBatis、MySQL、Redis 等技术栈实现，当前已经包含认证、发布、互动计数、关注关系、搜索、对象存储上传和 RAG 问答等核心能力。

## 项目简介

这个后端主要解决以下几类问题：

- 用户注册、登录、刷新令牌、退出登录
- 验证码发送与校验
- 用户资料维护，包括头像上传
- 知文草稿创建、内容上传、元数据更新、发布、删除
- 点赞、收藏、关注等互动行为
- 首页 Feed 与“我的发布”列表
- Elasticsearch 搜索与建议词
- 基于向量检索的 RAG 问答

## 技术栈

- Java 21
- Spring Boot 3.2
- Spring Security + JWT
- MyBatis
- MySQL 8
- Redis
- Kafka
- Caffeine
- Elasticsearch
- Spring AI
- 阿里云 OSS
- Canal

## 主要模块

`src/main/java/com/tongji` 下按业务拆分，核心模块如下：

- `auth`：认证、JWT、验证码、登录审计
- `profile`：个人资料、头像上传
- `knowpost`：知文草稿、内容确认、发布、详情、Feed
- `counter`：点赞、收藏等计数体系
- `relation`：关注/粉丝关系、Outbox 事件
- `storage`：对象存储预签名与上传相关接口
- `search`：Elasticsearch 检索与建议词
- `llm`：大模型能力与 RAG 问答
- `cache`：本地缓存与热点 Key 策略

## 目录说明

```text
KnowFlow_be
├── db/                         数据库初始化脚本
├── docs/                       接口与设计文档
├── src/main/java/com/tongji/   后端源码
├── src/main/resources/
│   ├── application.yml         主配置文件
│   ├── keys/                   JWT RSA 密钥
│   └── mapper/                 MyBatis XML
├── docker-compose.local.yml    本地依赖服务编排
├── pom.xml
└── README.md
```

## 运行环境

建议本地准备以下环境：

- JDK 21
- Maven 3.9+
- MySQL 8
- Redis 7

如果你要完整体验搜索、异步计数聚合、RAG 等能力，还需要：

- Kafka
- Elasticsearch
- 可用的模型 API Key
- 阿里云 OSS

## 本地快速启动

### 1. 初始化数据库

先创建数据库，例如：

```sql
CREATE DATABASE knowflow CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```

然后执行初始化脚本：

```bash
mysql -uroot -p knowflow < db/schema.sql
```

### 2. 配置本地密钥

项目支持从根目录下的 `.local-secrets.properties` 读取本地敏感配置，`application.yml` 已经通过：

```yaml
spring:
  config:
    import: optional:file:.local-secrets.properties
```

你可以新建一个本地配置文件，示例：

```properties
MYSQL_HOST=localhost
MYSQL_PORT=3306
MYSQL_DATABASE=knowflow
MYSQL_USERNAME=knowflow
MYSQL_PASSWORD=123456

REDIS_HOST=localhost
REDIS_PORT=6379

KAFKA_BOOTSTRAP_SERVERS=localhost:9092
SPRING_ELASTICSEARCH_URIS=http://localhost:9200

OSS_ENDPOINT=oss-cn-beijing.aliyuncs.com
OSS_ACCESS_KEY_ID=你的AccessKeyId
OSS_ACCESS_KEY_SECRET=你的AccessKeySecret
OSS_BUCKET=你的Bucket名称
OSS_PUBLIC_DOMAIN=

DEEPSEEK_API_KEY=
OPENAI_API_KEY=
OPENAI_BASE_URL=https://dashscope.aliyuncs.com/compatible-mode
OPENAI_EMBEDDING_MODEL=text-embedding-v4
OPENAI_EMBEDDING_DIMENSIONS=1536
```

说明：

- `.local-secrets.properties` 已在 `.gitignore` 中，不会默认提交
- 如果没有配置 OSS，头像上传和创作内容上传相关能力无法正常使用
- 如果没有配置 Elasticsearch / 模型能力，搜索与 RAG 相关功能可能不可用

### 3. 启动依赖服务

如果你本地已经装好了 MySQL、Redis、Kafka、Elasticsearch，可以直接使用。

如果你希望用 Docker 跑依赖，可以先查看：

- [docker-compose.local.yml](/Users/dxl/Desktop/KnowFlow_be/docker-compose.local.yml)

### 4. 启动项目

```bash
mvn spring-boot:run
```

或者：

```bash
mvn clean package
java -jar target/zhiguang-1.0-SNAPSHOT.jar
```

服务默认会启动在：

```text
http://localhost:8080
```

## 当前默认配置说明

根据 [application.yml](/Users/dxl/Desktop/KnowFlow_be/src/main/resources/application.yml)：

- MySQL 默认库名：`knowflow`
- Redis 默认地址：`localhost:6379`
- Kafka 默认地址：`localhost:9092`
- Elasticsearch 默认地址：`http://localhost:9200`
- JWT 采用 RSA 密钥对
- Access Token 有效期：15 分钟
- Refresh Token 有效期：7 天

## 开发提示

### 验证码发送

本地开发环境下，验证码发送通常是走日志输出方案，不一定会真正发到手机或邮箱。联调时请重点看后端日志和 Redis 中的验证码记录。

### 对象存储上传

当前项目的头像上传、创作内容上传依赖阿里云 OSS：

- 后端负责生成上传凭证或写入对象信息
- 前端直接上传文件到 OSS
- Bucket、地域、读写权限、CORS 都需要配置正确

### 搜索与 RAG

以下能力依赖额外组件：

- 搜索：Elasticsearch
- 向量检索：Spring AI + Elasticsearch Vector Store
- 问答：模型 API Key

如果只是先跑通基础业务，可以先不启用这部分完整能力。

## 接口与设计文档

仓库内已经提供了若干文档，可以直接查看：

- [db/schema.sql](/Users/dxl/Desktop/KnowFlow_be/db/schema.sql)
- [docs/API接口.md](/Users/dxl/Desktop/KnowFlow_be/docs/API接口.md)
- [docs/API接口文档.md](/Users/dxl/Desktop/KnowFlow_be/docs/API接口文档.md)
- [docs/API接口文档_knowpost.md](/Users/dxl/Desktop/KnowFlow_be/docs/API接口文档_knowpost.md)
- [docs/API接口文档_用户关系.md](/Users/dxl/Desktop/KnowFlow_be/docs/API接口文档_用户关系.md)
- [docs/API接口文档_计数.md](/Users/dxl/Desktop/KnowFlow_be/docs/API接口文档_计数.md)
- [docs/用户关系设计方案.md](/Users/dxl/Desktop/KnowFlow_be/docs/用户关系设计方案.md)
- [docs/计数系统设计方案.md](/Users/dxl/Desktop/KnowFlow_be/docs/计数系统设计方案.md)

## 前端说明

当前仓库是后端仓库。前端项目在你的本地是单独目录 `KnowFlow_fe`，如果准备上传到 GitHub，建议：

- 前后端分成两个仓库分别维护；或者
- 新建一个总仓库，把 `KnowFlow_be` 和 `KnowFlow_fe` 一起纳入

## 注意事项

- 不要把真实的 OSS、模型、数据库密码直接提交到 GitHub
- 如果你曾经泄露过阿里云 AccessKey，建议先去阿里云控制台停用旧 Key，再使用新 Key
- 提交代码前，建议再次确认 `.local-secrets.properties` 没有被纳入版本控制

## License

如需开源发布，建议你补充自己的 License 文件后再公开仓库。
