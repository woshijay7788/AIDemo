# AGENTS.md - AI 代理开发指南

## 项目概述
这是一个 Java SpringBoot AI 演示项目。

## 构建、代码检查和测试命令

### Java/Maven 项目

> 注意：本项目使用 Java 8 + Spring Boot 2.7.18

```bash
# 安装依赖
mvn clean install

# 编译
mvn compile

# 运行所有测试
mvn test

# 运行单个测试类
mvn test -Dtest=UserServiceTest

# 运行单个测试方法
mvn test -Dtest=UserServiceTest#testShouldReturnUserById

# 运行多个测试类（逗号分隔）
mvn test -Dtest=UserServiceTest,OrderServiceTest

# 使用通配符运行测试
mvn test -Dtest=*ServiceTest

# 运行应用
mvn spring-boot:run

# 打包
mvn clean package

# 跳过测试打包
mvn clean package -DskipTests
```

### 常用命令
```bash
# 查看依赖树
mvn dependency:tree

# 代码格式检查
mvn checkstyle:check

# 跳过检查打包
mvn clean package -DskipTests -Dcheckstyle.skip=true

# 运行 verify 阶段（包括所有检查）
mvn verify

# 调试模式运行应用
mvn spring-boot:run -Dspring-boot.run.jvmArguments="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
```

## 代码风格指南

### 导入规则
- 使用 IDE 自动导入功能
- 分组导入顺序：java 包 → javax 包 → 第三方库 → 本项目
- 不要使用通配符导入（如 `import java.util.*`）
- 静态导入放在最后

### 代码格式化
- 使用 4 个空格缩进（不要使用 Tab）
- 最大行长度：120 个字符
- 大括号风格：K&R 风格（左括号不换行）
- 所有文件末尾必须添加换行符
- 操作符两侧添加空格：`int a = b + c;`
- 逗号后加空格：`method(a, b, c)`

### 类型注解
- 为所有方法参数和返回值使用显式类型
- 为所有公共 API 编写 Javadoc
- 避免使用原始类型，使用泛型
- 推荐使用 `List<String>` 而非 `ArrayList<String>`

### 命名约定
- **类/接口**：PascalCase（例如 `UserService`、`RestController`）
- **方法/变量**：camelCase（例如 `getUserById`、`userName`）
- **常量**：SCREAMING_SNAKE_CASE（例如 `MAX_RETRY_COUNT`）
- **包名**：全小写（例如 `com.aidemo.controller`）
- **枚举**：PascalCase，枚举值也是 PascalCase（例如 `Status.ACTIVE`）

### 错误处理
- 使用特定的异常类型，而非捕获通用的 `Exception`
- 在错误消息中包含上下文信息
- 使用日志记录异常（不要只打印堆栈）
- 为领域特定错误使用自定义异常
- 不要捕获并吞掉异常（至少要记录日志）
- 使用 `try-with-resources` 管理资源

### 通用最佳实践
- 类控制在 200 行以内
- 方法控制在 30 行以内
- 单一职责原则：每个类/方法只做一件事
- 为所有公共类和方法编写 Javadoc
- 使用有意义的变量名，避免 `a`, `b`, `temp` 等
- 避免魔法数字——使用命名常量
- 提前返回以减少嵌套（卫语句模式）
- 使用 Optional 避免空指针

### SpringBoot 特定规范
- Controller 层只处理请求/响应，业务逻辑放在 Service 层
- 使用构造器注入（推荐 @RequiredArgsConstructor 或构造器）
- 配置类使用 `@Configuration` 注解
- 使用 `@Service` 注解标记服务层
- 使用 `@RestController` 注解 REST API
- 接口参数使用 `@RequestBody` 接收 JSON
- 使用 `@GetMapping`、`@PostMapping` 等替代 `@RequestMapping`

### 日志规范
- 使用 SLF4J 日志框架
- 日志级别：ERROR > WARN > INFO > DEBUG > TRACE
- 不要使用 `System.out.println()`
- 日志消息使用占位符：`log.info("User {} not found", userId)`

### Git 约定
- 使用约定式提交消息：`feat:`、`fix:`、`docs:`、`refactor:`、`test:`
- 提交信息格式：`<类型>: <简短描述>`（50 字符内）
- 保持提交原子性和专注性
- 编写描述性的 PR 标题和描述
- 永远不要提交密钥、凭证或敏感信息
- 不要提交 `application.yml` 中的敏感配置

### 测试指南
- 测试行为，而非实现细节
- 使用描述性的测试方法名：`testShouldReturnUserWhenValidIdProvided`
- 单元测试使用 JUnit 5
- 使用 `@SpringBootTest` 进行集成测试
- 使用 `@WebMvcTest` 进行 Controller 测试
- 使用 `@DataJpaTest` 进行 Repository 测试
- Mock 外部依赖（使用 Mockito）
- 测试边缘情况和错误条件
- 保持测试快速（单元测试应毫秒级完成）

## 在此仓库中工作

1. 编写代码之前，先了解需求
2. 遵循代码库中的现有模式
3. 提交前确保代码可以编译通过：`mvn compile`
4. 为新功能编写测试
5. 保持更改专注和最小化
6. 运行相关测试确保没有破坏现有功能

## 项目结构
```
src/
├── main/
│   ├── java/com/aidemo/
│   │   ├── controller/   # REST API 控制器
│   │   ├── service/      # 业务逻辑层
│   │   ├── repository/   # 数据访问层
│   │   ├── model/        # 实体类
│   │   ├── dto/          # 数据传输对象
│   │   └── exception/   # 自定义异常
│   └── resources/
│       ├── application.yml  # 应用配置
│       └── static/          # 静态资源
└── test/
    └── java/               # 测试代码
```