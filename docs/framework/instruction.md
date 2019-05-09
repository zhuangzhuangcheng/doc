## 框架及工具

### 服务端

- Spring cloud (待完成)
- Spring boot 
- Mybatis-plus
- Spring security (待完成)
- OAuth (待完成)
- Fastjson
- swagger
- Maven
- Git

## 模块

- tools-abf-service  ABFrame 服务
- tools-asf-service  项目脚手架服务
- toos-core 通用方法工具类
- tools-model 基础模型
- tools-parent 项目结构和依赖管理
- tools-starter-cors 跨域能力starter
- tools-starter-persistence 项目DAO能力starter
- tools-starter-swagger API文档starter
- tools-starter-log(待完成)  操作日志能力starter

## 具体实现功能

### tools-abf-service

#### 校验

使用 Hibernate Validation 在controller接口处对请求参数校验，通过抽象请求参数，使用注解形式约束请求，再通过统一异常拦截返回约束信息

#### 序列化

使用fastJson对请求和响应的数据序列化和反序列化，配合mybatis-plus能力实现通用枚举的自动注入

#### 乐观锁、逻辑删除源、公共字段填充

利用mybatis-plus 的能力实现，通过对基础sql的注入实现

#### 异常处理

使用Spring框架中的ExceptionHander拦截异常返回信息

### tools-starter-log

#### 记录操作日志

通过对controller方法使用指定注解，配合aop获取操作信息，数据记录通过aop拦截dao中所有增删改方法，获取数据变动记录。一次请求的数据通过ThreadLocal记录，在业务处理完成后使用异步线程池，在指定时间后异步持久化操作记录