# 苍穹外卖 (Sky Take-Out)

基于 Spring Boot 的餐饮外卖管理平台后端工程。

---

## 目录

- [项目架构](#项目架构)
- [技术栈](#技术栈)
- [模块说明](#模块说明)
  - [sky-common —— 公共模块](#sky-common--公共模块)
  - [sky-pojo —— 数据模型模块](#sky-pojo--数据模型模块)
  - [sky-server —— 服务端模块](#sky-server--服务端模块)
- [快速启动](#快速启动)
- [接口文档](#接口文档)

---

## 项目架构

```
sky-take-out (父工程)
├── sky-common          —— 公共组件：常量、工具类、异常体系、配置属性、统一返回结果
├── sky-pojo            —— 数据模型：实体类(Entity)、数据传输对象(DTO)、视图对象(VO)
└── sky-server          —— 服务端：Controller 层、Service 层、Mapper 层、配置、拦截器
```

采用经典的三层架构（Controller → Service → Mapper），基于 Maven 多模块构建。

### 请求处理流程

```
客户端 HTTP 请求
    │
    ▼
JwtTokenAdminInterceptor (JWT 令牌校验)
    │
    ▼
EmployeeController (Controller 层 — 接收请求、响应结果)
    │
    ▼
EmployeeService / EmployeeServiceImpl (Service 层 — 业务逻辑)
    │
    ▼
EmployeeMapper (Mapper 层 — MyBatis 数据库操作)
    │
    ▼
MySQL 数据库
```

---

## 技术栈

| 技术              | 作用                |
| ----------------- | ------------------- |
| Spring Boot 2.7.3 | 应用框架            |
| MyBatis + PageHelper | ORM 与分页        |
| MySQL             | 数据库              |
| Druid             | 数据库连接池        |
| Redis + Spring Cache | 缓存             |
| Spring WebSocket  | 即时通讯            |
| Knife4j (Swagger) | 接口文档            |
| JWT (jjwt)        | 令牌认证            |
| Lombok            | 代码简化            |
| FastJSON          | JSON 处理           |
| Apache HttpClient | HTTP 请求工具         |
| 阿里云 OSS SDK    | 文件存储            |
| 微信支付 SDK      | 微信支付            |
| Apache POI        | Excel 报表导出      |

---

## 模块说明

### sky-common —— 公共模块

提供全项目通用的基础组件，被 `sky-server` 和 `sky-pojo` 依赖。

#### 常量类 (`com.sky.constant`)

| 文件                            | 说明                                 |
| ------------------------------- | ------------------------------------ |
| `AutoFillConstant.java`         | 公共字段自动填充常量（创建/更新时间、创建/更新用户） |
| `JwtClaimsConstant.java`        | JWT 载荷中的字段名常量（empId、userId、phone 等） |
| `MessageConstant.java`          | 业务提示信息常量（密码错误、账号不存在等） |
| `PasswordConstant.java`         | 密码相关常量（默认密码 123456）        |
| `StatusConstant.java`           | 状态常量（ENABLE = 1 启用，DISABLE = 0 禁用） |

#### 上下文 (`com.sky.context`)

| 文件                       | 说明                                       |
| -------------------------- | ------------------------------------------ |
| `BaseContext.java`         | 基于 ThreadLocal 的线程上下文，存储当前登录用户 ID |

#### 枚举 (`com.sky.enumeration`)

| 文件                  | 说明                                   |
| --------------------- | -------------------------------------- |
| `OperationType.java`  | 数据库操作类型枚举（UPDATE / INSERT），用于 AOP 自动填充 |

#### 异常体系 (`com.sky.exception`)

所有异常继承自 `BaseException`（继承 `RuntimeException`），由全局异常处理器统一捕获。

| 文件                              | 说明                     |
| --------------------------------- | ------------------------ |
| `BaseException.java`              | 业务异常基类             |
| `AccountLockedException.java`     | 账号被锁定异常           |
| `AccountNotFoundException.java`   | 账号不存在异常           |
| `PasswordErrorException.java`     | 密码错误异常             |
| `PasswordEditFailedException.java` | 密码修改失败异常         |
| `LoginFailedException.java`       | 登录失败异常             |
| `UserNotLoginException.java`      | 用户未登录异常           |
| `AddressBookBusinessException.java` | 地址簿业务异常         |
| `ShoppingCartBusinessException.java` | 购物车业务异常       |
| `OrderBusinessException.java`     | 订单业务异常             |
| `DeletionNotAllowedException.java` | 不允许删除操作异常     |
| `SetmealEnableFailedException.java` | 套餐启用失败异常       |

#### Jackson JSON 配置

| 文件                        | 说明                                                   |
| --------------------------- | ------------------------------------------------------ |
| `JacksonObjectMapper.java`  | 自定义 ObjectMapper，配置 LocalDateTime/LocalDate/LocalTime 的序列化/反序列化格式 |

#### 配置属性类 (`com.sky.properties`)

通过 `@ConfigurationProperties` 绑定 `application.yml` 中的配置前缀。

| 文件                    | 绑定前缀        | 说明                         |
| ----------------------- | --------------- | ---------------------------- |
| `AliOssProperties.java` | `sky.alioss`    | 阿里云 OSS 配置（密钥、Bucket） |
| `JwtProperties.java`    | `sky.jwt`       | JWT 配置（密钥、过期时间、令牌名） |
| `WeChatProperties.java` | `sky.wechat`    | 微信支付配置（商户号、证书等）   |

#### 统一返回结果 (`com.sky.result`)

| 文件               | 说明                                                    |
| ------------------ | ------------------------------------------------------- |
| `Result.java`      | 后端统一返回结果，包含 code（1 成功 / 0 失败）、msg、data，提供 success() / error() 静态工厂 |
| `PageResult.java`  | 分页查询结果封装，包含 total（总记录数）和 records（当前页数据） |

#### 工具类 (`com.sky.utils`)

| 文件                 | 说明                                                |
| -------------------- | --------------------------------------------------- |
| `JwtUtil.java`       | JWT 令牌生成与解析工具，使用 HS256 算法               |
| `HttpClientUtil.java`| HTTP 请求工具，封装 GET/POST/POST-JSON 请求           |
| `AliOssUtil.java`    | 阿里云 OSS 文件上传工具                              |
| `WeChatPayUtil.java` | 微信支付工具（JSAPI 下单、小程序支付、申请退款）       |

---

### sky-pojo —— 数据模型模块

存放所有与数据交互相关的 Java Bean，不含业务逻辑。

#### 实体类 (`com.sky.entity`)

对应数据库表结构，使用 `@Data` + `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor` 简化。

| 文件                | 对应表              | 说明                                |
| ------------------- | ------------------ | ----------------------------------- |
| `Employee.java`     | `employee`         | 员工（管理员），含用户名、密码、状态、手机号等 |
| `Category.java`     | `category`         | 分类（1 菜品分类 / 2 套餐分类）          |
| `Dish.java`         | `dish`             | 菜品，含名称、价格、分类、图片、描述      |
| `DishFlavor.java`   | `dish_flavor`      | 菜品口味（如"辣度"、"规格"）           |
| `Setmeal.java`      | `setmeal`          | 套餐，含分类、价格、状态               |
| `SetmealDish.java`  | `setmeal_dish`     | 套餐-菜品关联（多对多）                |
| `Orders.java`       | `orders`           | 订单，含状态（1-6）、支付方式、金额、地址等，内置状态常量 |
| `OrderDetail.java`  | `order_detail`     | 订单明细（一个订单包含多个菜品）        |
| `ShoppingCart.java` | `shopping_cart`    | 购物车                             |
| `AddressBook.java`  | `address_book`     | 用户地址簿                          |
| `User.java`         | `user`             | C 端微信用户                         |

#### DTO (`com.sky.dto`)

数据传输对象，用于各层之间传递数据（如接收前端请求参数）。

**员工相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `EmployeeDTO.java`             | 员工新增/修改             |
| `EmployeeLoginDTO.java`        | 员工登录（用户名 + 密码） |
| `EmployeePageQueryDTO.java`    | 员工分页查询              |
| `PasswordEditDTO.java`         | 员工修改密码              |

**分类相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `CategoryDTO.java`            | 分类新增/修改              |
| `CategoryPageQueryDTO.java`   | 分类分页查询               |

**菜品相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `DishDTO.java`                 | 菜品新增/修改（含口味列表）|
| `DishPageQueryDTO.java`        | 菜品分页查询               |

**套餐相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `SetmealDTO.java`              | 套餐新增/修改（含菜品列表）|
| `SetmealPageQueryDTO.java`     | 套餐分页查询               |

**购物车相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `ShoppingCartDTO.java`         | 添加购物车（菜品/套餐 ID） |

**订单相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `OrdersSubmitDTO.java`         | 用户提交订单               |
| `OrdersDTO.java`               | 订单数据传输               |
| `OrdersPageQueryDTO.java`      | 订单分页查询               |
| `OrdersPaymentDTO.java`        | 订单支付                  |
| `OrdersCancelDTO.java`         | 取消订单                  |
| `OrdersConfirmDTO.java`        | 接单                     |
| `OrdersRejectionDTO.java`      | 拒单                     |

**用户/数据相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `UserLoginDTO.java`            | 微信用户登录（code）       |
| `DataOverViewQueryDTO.java`    | 数据概览查询时间区间        |
| `GoodsSalesDTO.java`           | 商品销量排行               |

#### VO (`com.sky.vo`)

视图对象，用于封装返回给前端的响应数据。

**登录相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `EmployeeLoginVO.java`         | 员工登录响应（ID + token） |
| `UserLoginVO.java`             | 用户登录响应（ID + openid + token） |

**菜品/套餐展示：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `DishVO.java`                  | 菜品展示（含分类名、口味）|
| `DishItemVO.java`              | 套餐内的菜品项             |
| `DishOverViewVO.java`          | 菜品总览（启售/停售数量） |
| `SetmealVO.java`               | 套餐展示（含分类名、关联菜品）|
| `SetmealOverViewVO.java`       | 套餐总览（启售/停售数量） |

**订单相关：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `OrderSubmitVO.java`           | 提交订单响应              |
| `OrderVO.java`                 | 订单详情（继承 Orders + 菜品信息）|
| `OrderPaymentVO.java`          | 支付参数（给小程序调起支付）|
| `OrderOverViewVO.java`         | 订单概览（各状态数量）     |
| `OrderStatisticsVO.java`       | 订单统计（待接单/待派送/派送中）|

**报表/数据：**
| 文件                           | 说明                     |
| ------------------------------ | ------------------------ |
| `TurnoverReportVO.java`        | 营业额统计报表            |
| `OrderReportVO.java`           | 订单统计报表              |
| `UserReportVO.java`            | 用户统计报表              |
| `SalesTop10ReportVO.java`      | 销量 Top10 报表           |
| `BusinessDataVO.java`          | 数据概览（营业额、订单数、客单价、新用户）|

---

### sky-server —— 服务端模块

包含所有业务逻辑和 Web 层代码，是应用的入口。

#### 启动类

| 文件                    | 说明                                          |
| ----------------------- | --------------------------------------------- |
| `SkyApplication.java`   | Spring Boot 启动类，含 `@EnableTransactionManagement` 开启事务 |

#### 配置 (`com.sky.config`)

| 文件                        | 说明                                               |
| --------------------------- | -------------------------------------------------- |
| `WebMvcConfiguration.java`  | Web MVC 配置：注册拦截器、Knife4j 接口文档 Docket、静态资源映射 |

#### 拦截器 (`com.sky.interceptor`)

| 文件                            | 说明                                                                 |
| ------------------------------- | -------------------------------------------------------------------- |
| `JwtTokenAdminInterceptor.java` | 管理员端 JWT 令牌校验拦截器，拦截 `/admin/**` 路径（除登录外），验证令牌有效性，失败返回 401 |

#### 全局异常处理 (`com.sky.handler`)

| 文件                            | 说明                                             |
| ------------------------------- | ------------------------------------------------ |
| `GlobalExceptionHandler.java`   | `@RestControllerAdvice` 全局异常处理器，捕获 `BaseException` 返回统一错误 Result |

#### Controller 层

| 文件                              | 说明                   |
| --------------------------------- | ---------------------- |
| `EmployeeController.java`         | 员工管理（登录、退出）   |

当前实现了：
- `POST /admin/employee/login` — 员工登录（校验账号密码、生成 JWT 令牌）
- `POST /admin/employee/logout` — 员工退出

#### Service 层

| 文件                     | 说明                                     |
| ------------------------ | ---------------------------------------- |
| `EmployeeService.java`   | 员工服务接口                              |
| `EmployeeServiceImpl.java` | 员工服务实现 — 登录逻辑：查用户 → 比对密码 → 校验状态 |

#### Mapper 层（MyBatis）

| 文件                  | 说明                                    |
| --------------------- | --------------------------------------- |
| `EmployeeMapper.java` | 员工数据库操作（`@Select` 按用户名查询） |

#### MyBatis XML

| 文件                          | 说明           |
| ----------------------------- | -------------- |
| `EmployeeMapper.xml`          | SQL 映射文件（当前为空的脚手架） |

---

## 快速启动

### 前置要求

- JDK 17+
- Maven 3.6+
- MySQL 8.0+
- Redis (部分功能需要)

### 1. 创建数据库

```sql
CREATE DATABASE sky_take_out;
```

### 2. 修改配置

编辑 `sky-server/src/main/resources/application-dev.yml`：

```yaml
sky:
  datasource:
    host: localhost
    port: 3306
    database: sky_take_out
    username: root
    password: 你的MySQL密码
```

### 3. 编译 & 运行

```bash
# 编译整个项目
mvn clean compile

# 安装到本地 Maven 仓库（确保子模块依赖可用）
mvn install -DskipTests

# 启动应用
mvn spring-boot:run -pl sky-server
```

或者直接在 IDEA 中打开项目，运行 `SkyApplication.main()`。

### 4. 访问

- 服务地址：`http://localhost:8080`
- 接口文档：`http://localhost:8080/doc.html`（Knife4j Swagger UI）

---

## 接口文档

启动后访问 [http://localhost:8080/doc.html](http://localhost:8080/doc.html) 即可查看所有 API 接口文档并进行在线调试。

文档由 Knife4j（Swagger）自动生成，扫描 `com.sky.controller` 包下的所有 `@RestController`。

---

## 注意

- `.gitignore` 中忽略了 `*Test.java` 和 `**/test/`，测试文件不会被 Git 追踪
- 默认启用 `dev` profile（`spring.profiles.active: dev`），可根据环境添加不同的配置文件
- JWT 默认密钥为 `itcast`，过期时间 7200000ms（2 小时），仅用于开发环境
