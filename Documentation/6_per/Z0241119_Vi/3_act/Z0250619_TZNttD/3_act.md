## Z025/06/19（四）

“HAFJ OSS” 通常指的是 **Hitachi Application Framework for Java Open Source Software（HAFJ OSS）**，这是 **日立公司推出的一套基于 Java 的企业级开发框架**，用于构建 Web 应用或企业信息系统。

以下是对 HAFJ OSS 的安装、作用和安装注意点的总结：

---

## 🧩 一、HAFJ OSS 作用

HAFJ OSS 是一个构建企业级 Web 应用程序的基础框架，整合了多个开源组件，目的是提高开发效率、保证系统稳定性。

### 核心功能：

* 支持 **MVC 架构**（通常与 Struts/Spring MVC 搭配）
* 集成 **日志管理（log4j/slf4j）**
* 数据访问层封装（可能结合 MyBatis 或 JDBC）
* 国际化（i18n）和多语言支持
* 与 **COBOL 主机系统的协作（常见于日立主机系统）**
* 安全框架（权限控制、会话管理）

---

## 🛠 二、安装步骤

HAFJ OSS 通常作为 Eclipse 项目的开发框架使用，安装方式如下：

### 1. 环境准备

* **Java JDK**：建议使用 Java 8 或 11，视项目而定
* **Eclipse IDE for Enterprise Java**（支持 JSP/Servlet）
* **JBoss/WildFly 或 Tomcat**：作为 Web 应用服务器
* **MySQL/PostgreSQL/Oracle** 数据库（视项目需求）
* **Maven/Ant**（根据项目构建方式）

### 2. 下载 HAFJ OSS

* 从日立官网下载 HAFJ OSS 相关包（可能需要企业内部账号）
* 解压后查看其中的：

  * `hafj-oss-core.jar`
  * 示例项目模板
  * 配套文档（如 `README`, `install-guide.pdf`）

### 3. 建立 Eclipse 项目

* 新建 **Dynamic Web Project** 或 **Maven Project**
* 将 HAFJ OSS 依赖 JAR 包导入到 `lib` 目录或通过 Maven 添加依赖
* 按照样板项目配置：

  * `web.xml`
  * `struts-config.xml` 或 `applicationContext.xml`
  * `log4j.properties`
  * JSP 页面模板和 action 类

### 4. 配置数据库连接

* 修改 `db.properties` 或 `applicationContext.xml` 中的数据源配置
* 测试连接数据库是否正常

### 5. 部署到服务器

* 在 Eclipse 中配置 JBoss/Tomcat，启动部署应用
* 访问 `http://localhost:8080/项目名/index.jsp` 进行初步测试

---

## ⚠️ 三、安装注意事项

| 项目           | 注意点                                                |
| ------------ | -------------------------------------------------- |
| **兼容性**      | 确保 HAFJ OSS 与 JDK、Servlet 容器版本兼容                   |
| **字符编码**     | 推荐使用 UTF-8 编码，特别是在多语言系统中                           |
| **目录结构**     | 遵守框架目录规范（如：action、form、service、dao）                |
| **依赖冲突**     | 避免 log4j 等版本冲突，使用统一版本管理                            |
| **配置文件路径**   | 所有配置文件需放在 `WEB-INF/classes` 或 `src/main/resources` |
| **JSP 编码声明** | 确保 JSP 页头声明编码为 UTF-8 以避免乱码                         |
| **部署权限**     | 某些主机集成系统可能需要特殊部署权限（特别是 Hitachi CO**BOL** 系统）       |
| **文档阅读**     | 仔细阅读 HAFJ OSS 提供的 PDF 文档和示例说明，了解其特定约定              |

---

如果你是企业项目开发人员，安装后建议先运行官方提供的 Sample Project，确认环境正常，再进行业务逻辑开发。

---

如果你有具体的操作系统（如 Windows/Linux）、目标服务器（Tomcat/JBoss）或项目类型（是否对接 COBOL）的需求，也可以告诉我，我可以提供具体配置示例。
