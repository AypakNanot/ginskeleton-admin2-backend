# GinSkeleton-Admin2 Backend

> 基于 GinSkeleton v1.5.xx 开发的企业级后台管理系统后端服务

---

## 目录

- [项目简介](#项目简介)
- [技术栈](#技术栈)
- [项目结构](#项目结构)
- [快速开始](#快速开始)
- [核心功能模块](#核心功能模块)
- [开发指南](#开发指南)
  - [添加新功能模块](#添加新功能模块)
  - [添加 API 路由](#添加-api-路由)
  - [创建数据模型](#创建数据模型)
  - [编写 Service 层](#编写-service-层)
  - [表单验证器](#表单验证器)
- [配置文件说明](#配置文件说明)
- [数据库设计](#数据库设计)
- [权限控制](#权限控制)
- [常见问题](#常见问题)

---

## 项目简介

GinSkeleton-Admin2 是一个基于 Go + Gin 框架开发的企业级后台管理系统后端服务。项目采用经典的分层架构设计，集成了用户管理、角色权限、菜单管理、按钮权限控制、文件上传、WebSocket 等企业常用功能。

### 主要特性

- **多应用入口**: 支持后台管理系统 (web) 和门户 API 系统 (api) 独立部署
- **完善的权限体系**: 基于 Casbin 的 RBAC 权限控制，支持菜单、按钮级别授权
- **JWT 认证**: 支持 Token 刷新机制，可配置 Redis 缓存
- **多数据库支持**: MySQL、SQL Server、PostgreSQL，支持读写分离
- **日志系统**: 基于 Zap 的结构化日志，支持日志轮转
- **WebSocket**: 内置 WebSocket 支持，可配置启动
- **AOP 思想**: 支持控制器前置/后置回调
- **观察者模式**: 内置事件订阅/发布机制

---

## 技术栈

| 类别 | 技术 |
|------|------|
| Web 框架 | Gin v1.10.1 |
| ORM | GORM v1.30.0 |
| 权限控制 | Casbin v2.105.0 |
| 认证 | JWT (dgrijalva/jwt-go) |
| 日志 | Zap v1.27.0 |
| 配置 | Viper v1.20.1 |
| 验证码 | dchest/captcha v1.1.0 |
| WebSocket | gorilla/websocket v1.5.3 |
| 消息队列 | RabbitMQ (amqp091-go) |
| 缓存 | Redis (gomodule/redigo) |
| 雪花算法 | 自定义实现 |
| 参数验证 | go-playground/validator/v10 |

---

## 项目结构

```
ginskeleton-admin2-backend/
├── app/                          # 应用核心代码
│   ├── aop/                      # AOP 前置/后置回调
│   │   └── users/
│   ├── core/                     # 核心功能
│   │   ├── container/            # 依赖注入容器
│   │   ├── destroy/              # 程序退出资源清理
│   │   └── event_manage/         # 事件管理
│   ├── global/                   # 全局定义
│   │   ├── consts/               # 常量定义
│   │   ├── my_errors/            # 错误定义
│   │   └── variable/             # 全局变量
│   ├── http/                     # HTTP 层
│   │   ├── controller/           # 控制器层
│   │   │   ├── api/              # API 控制器
│   │   │   ├── captcha/          # 验证码控制器
│   │   │   ├── web/              # Web 后台控制器
│   │   │   └── websocket/        # WebSocket 控制器
│   │   ├── middleware/           # 中间件
│   │   │   ├── authorization/    # 认证授权中间件
│   │   │   ├── cors/             # 跨域中间件
│   │   │   └── my_jwt/           # JWT 中间件
│   │   └── validator/            # 表单验证器
│   │       ├── api/              # API 验证器
│   │       ├── common/           # 通用验证器
│   │       └── web/              # Web 验证器
│   ├── model/                    # 数据模型层
│   │   ├── auth/                 # 权限相关模型
│   │   ├── users/                # 用户模型
│   │   └── province_city/        # 省市区模型
│   └── service/                  # 业务逻辑层
│       ├── auth_post_members/
│       ├── auth_system_menu/
│       ├── upload_file/
│       └── sys_log_hook/
├── bootstrap/                    # 启动初始化
│   └── init.go
├── cmd/                          # 应用入口
│   ├── api/                      # API 门户入口
│   ├── web/                      # 后台管理入口
│   └── cli/                      # 命令行入口
├── command/                      # CLI 命令定义
├── config/                       # 配置文件
│   ├── config.yml                # 主配置文件
│   └── gorm_v2.yml               # 数据库配置
├── database/                     # 数据库相关文件
├── routers/                      # 路由定义
│   ├── api.go                    # API 路由
│   └── web.go                    # Web 路由
├── storage/                      # 存储目录
│   ├── logs/                     # 日志文件
│   └── app/uploaded/             # 上传文件
├── public/                       # 静态资源
├── test/                         # 测试文件
├── go.mod                        # Go 模块定义
└── main.go                       # 默认入口
```

---

## 快速开始

### 环境要求

- Go >= 1.24
- MySQL >= 5.7 (推荐 8.0)
- Redis >= 3.0 (可选，用于缓存 Token)
- RabbitMQ (可选，用于消息队列)

### 安装步骤

1. **克隆项目**
```bash
git clone <repository-url>
cd ginskeleton-admin2-backend
```

2. **安装依赖**
```bash
go mod download
```

3. **配置数据库**

编辑 `config/gorm_v2.yml`，修改数据库连接信息：
```yaml
Gormv2:
  UseDbType: "mysql"
  Mysql:
    IsInitGlobalGormMysql: 1
    Write:
      Host: "127.0.0.1"
      DataBase: "db_ginskeleton2"
      Port: 3306
      User: "root"
      Pass: "YourDbPassword"  # 修改为你的密码
      Charset: "utf8"
```

4. **配置应用**

编辑 `config/config.yml`，修改关键配置：
```yaml
AppDebug: true  # 开发环境设为 true

HttpServer:
  Web:
    Port: ":22001"  # 后台管理端口
  Api:
    Port: ":22002"  # API 门户端口

Token:
  JwtTokenSignKey: "goskeleton"  # JWT 签名密钥
```

5. **导入数据库**

导入 `database/` 目录中的 SQL 文件到 MySQL

6. **运行项目**

启动后台管理服务：
```bash
go run cmd/web/main.go
```

启动 API 门户服务：
```bash
go run cmd/api/main.go
```

7. **访问服务**

- 后台管理地址：http://localhost:22001
- API 门户地址：http://localhost:22002
- 健康检查：http://localhost:22001/

---

## 核心功能模块

### 1. 用户管理模块

| 接口 | 方法 | 描述 |
|------|------|------|
| /admin/users/list | GET | 用户列表查询 |
| /admin/users/create | POST | 新增用户 |
| /admin/users/edit | POST | 编辑用户 |
| /admin/users/destroy | POST | 删除用户 |
| /admin/users/info | GET | 获取当前用户信息 |
| /admin/users/personal_info | GET | 获取个人信息 |
| /admin/users/login | POST | 用户登录 |
| /admin/users/refreshtoken | POST | 刷新 Token |

### 2. 权限管理模块

| 模块 | 描述 |
|------|------|
| 组织机构 | 公司/部门组织架构管理 |
| 岗位管理 | 岗位定义与维护 |
| 岗位成员 | 用户与岗位的关联 |
| 系统菜单 | 动态菜单配置 |
| 按钮权限 | 页面按钮级权限控制 |
| 菜单分配 | 为部门/岗位分配菜单权限 |

### 3. 系统功能

| 功能 | 描述 |
|------|------|
| 文件上传 | 支持图片、文档、视频等格式 |
| 验证码 | 图形验证码生成与校验 |
| WebSocket | 实时消息推送 |
| 日志系统 | 结构化日志记录 |
| 省市区数据 | 行政区划数据管理 |

---

## 开发指南

### 添加新功能模块

本项目采用经典的分层架构：**Controller → Validator → Service → Model**

以下以添加"文章管理"模块为例，说明如何添加新功能：

#### 步骤 1: 创建数据模型 (Model)

在 `app/model/article/` 目录创建模型：

```go
// app/model/article/article.go
package article

import (
    "github.com/gin-gonic/gin"
    "goskeleton/app/model"
    "goskeleton/app/utils/data_bind"
    "go.uber.org/zap"
    "goskeleton/app/global/variable"
)

// 创建模型工厂
func CreateArticleFactory(sqlType string) *ArticleModel {
    return &ArticleModel{BaseModel: model.BaseModel{DB: model.UseDbConn(sqlType)}}
}

type ArticleModel struct {
    model.BaseModel
    Title   string `json:"title"`
    Content string `json:"content"`
    Status  int    `json:"status"`
    AuthorId int64 `json:"author_id"`
}

func (a *ArticleModel) TableName() string {
    return "tb_article"
}

// 查询列表
func (a *ArticleModel) List(keyword string, limitStart, limitItems int) (int64, []ArticleModel) {
    // 实现查询逻辑
}

// 新增
func (a *ArticleModel) InsertData(c *gin.Context) bool {
    var tmp ArticleModel
    if err := data_bind.ShouldBindFormDataToModel(c, &tmp); err == nil {
        if res := a.Create(&tmp); res.Error == nil {
            return true
        }
        variable.ZapLog.Error("文章新增出错", zap.Error(res.Error))
    }
    return false
}

// 更新
func (a *ArticleModel) UpdateData(c *gin.Context) bool {
    // 实现更新逻辑
    return false
}

// 删除
func (a *ArticleModel) DeleteData(id int) bool {
    return a.Delete(a, id).Error == nil
}
```

#### 步骤 2: 创建业务服务层 (Service)

在 `app/service/article/` 目录创建服务：

```go
// app/service/article/article_service.go
package article

import (
    "github.com/gin-gonic/gin"
    "goskeleton/app/model/article"
)

func CreateArticleService() *ArticleService {
    return &ArticleService{}
}

type ArticleService struct{}

// 获取文章列表
func (s *ArticleService) GetList(keyword string, page, limit int) (int64, []article.ArticleModel) {
    limitStart := (page - 1) * limit
    return article.CreateArticleFactory("").List(keyword, limitStart, limit)
}

// 创建文章
func (s *ArticleService) Create(c *gin.Context) bool {
    return article.CreateArticleFactory("").InsertData(c)
}

// 更新文章
func (s *ArticleService) Update(c *gin.Context) bool {
    return article.CreateArticleFactory("").UpdateData(c)
}

// 删除文章
func (s *ArticleService) Delete(id int) bool {
    return article.CreateArticleFactory("").DeleteData(id)
}
```

#### 步骤 3: 创建表单验证器 (Validator)

在 `app/http/validator/web/article/` 目录创建验证器：

```go
// app/http/validator/web/article/create.go
package article

import (
    "github.com/gin-gonic/gin"
    "goskeleton/app/global/consts"
    "goskeleton/app/utils/response"
)

// Create 参数验证与控制器调用
func Create(context *gin.Context) {
    // 获取并验证参数
    title := context.GetString(consts.ValidatorPrefix + "title")
    if title == "" {
        response.Fail(context, 400, "标题不能为空", "")
        return
    }

    content := context.GetString(consts.ValidatorPrefix + "content")

    // 调用控制器
    // (&Article{}).Store(context)
}
```

注册验证器到容器：

```go
// app/http/validator/common/register_validator/web_register_validator.go
func WebRegisterValidator() {
    // ... 其他注册

    // 注册文章验证器
    container.CreateContainersFactory().Set(
        consts.ValidatorPrefix+"ArticleCreate",
        &article.CreateValidator{},
    )
}
```

#### 步骤 4: 创建控制器 (Controller)

在 `app/http/controller/web/` 目录创建控制器：

```go
// app/http/controller/web/article_controller.go
package web

import (
    "github.com/gin-gonic/gin"
    "goskeleton/app/global/consts"
    "goskeleton/app/service/article"
    "goskeleton/app/utils/response"
)

type Article struct{}

// 文章列表
func (a *Article) List(context *gin.Context) {
    keyword := context.GetString(consts.ValidatorPrefix + "keyword")
    page := context.GetFloat64(consts.ValidatorPrefix + "page")
    limit := context.GetFloat64(consts.ValidatorPrefix + "limit")

    totalCounts, list := article.CreateArticleService().GetList(
        keyword, int(page), int(limit),
    )

    if totalCounts > 0 {
        response.Success(context, consts.CurdStatusOkMsg, gin.H{
            "count": totalCounts,
            "data":  list,
        })
    } else {
        response.Fail(context, consts.CurdSelectFailCode, consts.CurdSelectFailMsg, "")
    }
}

// 创建文章
func (a *Article) Store(context *gin.Context) {
    if article.CreateArticleService().Create(context) {
        response.Success(context, consts.CurdStatusOkMsg, "")
    } else {
        response.Fail(context, consts.CurdCreatFailCode, consts.CurdCreatFailMsg, "")
    }
}

// 更新文章
func (a *Article) Update(context *gin.Context) {
    if article.CreateArticleService().Update(context) {
        response.Success(context, consts.CurdStatusOkMsg, "")
    } else {
        response.Fail(context, consts.CurdUpdateFailCode, consts.CurdUpdateFailMsg, "")
    }
}

// 删除文章
func (a *Article) Destroy(context *gin.Context) {
    id := int(context.GetFloat64(consts.ValidatorPrefix + "id"))
    if article.CreateArticleService().Delete(id) {
        response.Success(context, consts.CurdStatusOkMsg, "")
    } else {
        response.Fail(context, consts.CurdDeleteFailCode, consts.CurdDeleteFailMsg, "")
    }
}
```

---

### 添加 API 路由

在 `routers/web.go` 中添加路由配置：

```go
// routers/web.go
func InitWebRouter() *gin.Engine {
    router := gin.Default()

    // ... 现有路由

    backend := router.Group("/admin/")
    backend.Use(authorization.CheckTokenAuth(), authorization.CheckCasbinAuth())
    {
        // ... 现有路由组

        // ===== 新增：文章管理路由 =====
        article := backend.Group("article/")
        {
            article.GET("list", validatorFactory.Create(consts.ValidatorPrefix+"ArticleList"))
            article.POST("create", validatorFactory.Create(consts.ValidatorPrefix+"ArticleCreate"))
            article.POST("edit", validatorFactory.Create(consts.ValidatorPrefix+"ArticleEdit"))
            article.POST("destroy", validatorFactory.Create(consts.ValidatorPrefix+"ArticleDestroy"))
        }
        // ============================
    }

    return router
}
```

---

### 创建数据模型

模型需要继承 `BaseModel` 以获得 GORM 基础能力：

```go
// app/model/base_model.go
package model

import (
    "gorm.io/gorm"
)

type BaseModel struct {
    ID        int64          `gorm:"primarykey" json:"id"`
    CreatedAt gorm.CreatedAt `json:"created_at"`
    UpdatedAt gorm.UpdatedAt `json:"updated_at"`
    DB        *gorm.DB       `gorm:"-" json:"-"`
}
```

使用数据库连接：
```go
// 使用默认数据库
model := CreateArticleFactory("")

// 或指定数据库类型
model := CreateArticleFactory("mysql")
```

---

### 编写 Service 层

Service 层封装业务逻辑，供 Controller 调用：

```go
// 标准 Service 结构
type ArticleService struct{}

func CreateArticleService() *ArticleService {
    return &ArticleService{}
}

// 业务方法
func (s *ArticleService) DoSomething(id int64) error {
    // 1. 调用 Model 层
    // 2. 处理业务逻辑
    // 3. 返回结果
    return nil
}
```

---

### 表单验证器

验证器采用结构体方式，实现 `CheckParams` 方法：

```go
// 验证器结构
type ArticleCreateValidator struct{}

// 实现验证接口
func (v *ArticleCreateValidator) CheckParams(context *gin.Context) {
    // 1. 参数验证
    title := context.PostForm("title")
    if title == "" {
        response.Fail(context, 400, "标题不能为空", "")
        return
    }

    // 2. 参数处理
    context.Set(consts.ValidatorPrefix+"title", title)

    // 3. 调用控制器
    (&Article{}).Store(context)
}
```

---

## 配置文件说明

### config/config.yml

主配置文件，包含以下模块：

| 配置项 | 描述 |
|--------|------|
| AppDebug | 调试模式开关 |
| HttpServer | HTTP 服务器配置（端口、跨域、代理） |
| Token | JWT Token 配置 |
| LoginPolicy | 登录安全策略 |
| Redis | Redis 配置 |
| Logs | 日志配置 |
| Websocket | WebSocket 配置 |
| SnowFlake | 雪花算法配置 |
| FileUploadSetting | 文件上传配置 |
| RabbitMq | RabbitMQ 配置 |
| Casbin | 权限模型配置 |
| Captcha | 验证码配置 |

### config/gorm_v2.yml

数据库配置，支持：

- MySQL
- SQL Server
- PostgreSQL
- 读写分离配置

---

## 数据库设计

### 核心数据表

| 表名 | 描述 |
|------|------|
| tb_users | 用户表 |
| tb_auth_access_tokens | Token 表 |
| tb_auth_system_menu | 系统菜单表 |
| tb_auth_organization_post | 组织机构/岗位表 |
| tb_auth_post_members | 岗位成员关联表 |
| tb_auth_menu_assign | 菜单分配表 |
| tb_auth_casbin_rule | Casbin 权限规则表 |
| tb_auth_button_cn_en | 按钮中英文名称表 |
| tb_province_city | 省市区数据表 |

---

## 权限控制

### Casbin 权限模型

项目使用 Casbin 进行权限控制，模型配置：

```conf
[request_definition]
r = sub, obj, act
[policy_definition]
p = sub, obj, act
[role_definition]
g = _ , _
[policy_effect]
e = some(where (p.eft == allow))
[matchers]
m = (g(r.sub, p.sub) || p.sub == "*" ) && keyMatch(r.obj , p.obj) && (r.act == p.act || p.act == "*")
```

### 权限中间件

```go
// 在路由中使用
backend.Use(
    authorization.CheckTokenAuth(),   // Token 验证
    authorization.CheckCasbinAuth(),  // Casbin 权限校验
)
```

---

## 常见问题

### Q1: 如何修改服务端口？

编辑 `config/config.yml`:
```yaml
HttpServer:
  Web:
    Port: ":22001"  # 修改此端口
```

### Q2: 如何连接 Redis?

编辑 `config/config.yml`:
```yaml
Redis:
  Host: "127.0.0.1"
  Port: 6379
  Auth: "your_password"
```

### Q3: Token 缓存如何开启？

编辑 `config/config.yml`:
```yaml
Token:
  IsCacheToRedis: 1  # 1=开启，0=关闭
```

### Q4: 如何添加新的中间件？

在 `app/http/middleware/` 目录创建中间件文件：

```go
// app/http/middleware/logger/logger.go
package logger

import "github.com/gin-gonic/gin"

func Logger() gin.HandlerFunc {
    return func(c *gin.Context) {
        // 中间件逻辑
        c.Next()
    }
}
```

在路由中注册：
```go
router.Use(logger.Logger())
```

### Q5: 如何自定义日志格式？

编辑 `config/config.yml`:
```yaml
Logs:
  TextFormat: "json"  # json 或 console
  TimePrecision: "millisecond"  # second 或 millisecond
```

---

## 更新日志

### v2.1.02 (2025-06-02)
- 依赖包更新至最新版
- 改进部分代码细节

---

## License

MIT License
