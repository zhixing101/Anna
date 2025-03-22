## Z025/03/22（六）

在 Eclipse 中使用 **JBoss 服务器 + MySQL 数据库** 创建 **Spring MVC + MyBatis + JSP** 项目，详细步骤如下：

---

## **1. 安装和配置 JBoss**
### **1.1 下载 JBoss**
- 访问 [JBoss EAP 官方网站](https://www.redhat.com/en/technologies/jboss-middleware/application-platform)
- 下载并解压到本地，例如：`C:\jboss-eap-7.4`

### **1.2 在 Eclipse 中添加 JBoss**
1. **安装 JBoss Tools 插件**
   - 打开 Eclipse，点击 **Help → Eclipse Marketplace**
   - 搜索 **JBoss Tools**，安装

2. **添加 JBoss 服务器**
   - **Window → Show View → Servers**
   - **右键 → New → Server**
   - 选择 **JBoss → JBoss EAP 7.x**
   - 设置 JBoss Home 目录为 `C:\jboss-eap-7.4`
   - 选择 JDK 版本（建议 JDK 11 或 17）

---

## **2. 创建新的 Web 项目**
1. **文件 → 新建 → Dynamic Web Project**
2. **项目名称**：`JBossSpringMVC`
3. **目标运行时**：选择 **JBoss EAP 7.x**
4. **动态 Web 模块版本**：选择 **3.1**（支持 Servlet 3.1）
5. **默认编码**：设置为 **UTF-8**
   - 点击 **Next → Next → Finish**

### **2.1 设置编码**
- **Window → Preferences → General → Workspace**
  - **Text file encoding** 选择 **UTF-8**
- **Window → Preferences → Web → JSP Files**
  - **Encoding** 选择 **UTF-8**
- **Window → Preferences → Java → Compiler**
  - **JDK 编码** 选择 **UTF-8**

---

## **3. 添加 Maven 支持**
1. 右键项目，选择 **Configure → Convert to Maven Project**
2. `pom.xml` 添加依赖：
```xml
<dependencies>
    <!-- Spring MVC -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-webmvc</artifactId>
        <version>5.3.30</version>
    </dependency>

    <!-- MyBatis -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.14</version>
    </dependency>

    <!-- MySQL 驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>

    <!-- JSTL -->
    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>jstl</artifactId>
        <version>1.2</version>
    </dependency>
</dependencies>
```
---

## **4. 配置 Spring MVC**
### **4.1 配置 `web.xml`**
`src/main/webapp/WEB-INF/web.xml`
```xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         version="3.1">
    <display-name>JBoss Spring MVC</display-name>

    <servlet>
        <servlet-name>spring</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring-servlet.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>spring</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <welcome-file-list>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

### **4.2 配置 Spring**
`src/main/webapp/WEB-INF/spring-servlet.xml`
```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/views/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
</beans>
```

---

## **5. 配置 MySQL**
### **5.1 配置 `db.properties`**
`src/main/resources/db.properties`
```properties
db.driver=com.mysql.cj.jdbc.Driver
db.url=jdbc:mysql://localhost:3306/yourdb
db.username=root
db.password=yourpassword
```

### **5.2 配置 MyBatis**
`src/main/java/com/example/project/config/MyBatisConfig.java`
```java
@Configuration
@MapperScan("com.example.project.mapper")
public class MyBatisConfig {
    @Bean
    public DataSource dataSource() throws IOException {
        Properties properties = new Properties();
        properties.load(getClass().getClassLoader().getResourceAsStream("db.properties"));
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(properties.getProperty("db.driver"));
        dataSource.setUrl(properties.getProperty("db.url"));
        dataSource.setUsername(properties.getProperty("db.username"));
        dataSource.setPassword(properties.getProperty("db.password"));
        return dataSource;
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSource dataSource) throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource);
        return sessionFactory.getObject();
    }
}
```

---

## **6. 创建 Controller**
`src/main/java/com/example/project/controller/HomeController.java`
```java
@Controller
public class HomeController {
    @RequestMapping("/")
    public String home() {
        return "index";
    }
}
```

---

## **7. 创建 JSP 页面**
### **7.1 `index.jsp`**
`src/main/webapp/WEB-INF/views/index.jsp`
```jsp
<%@ page contentType="text/html; charset=UTF-8" %>
<html>
<head><title>主页</title></head>
<body>
    <h1>欢迎访问 JBoss Spring MVC</h1>
</body>
</html>
```

---

## **8. 部署到 JBoss**
### **8.1 启动 JBoss**
在 Eclipse **Servers 视图**：
- 右键 **JBoss Server**
- 选择 **Start**

### **8.2 配置 JBoss 数据源**
1. 进入 `JBoss/bin/standalone.conf`，添加：
   ```sh
   JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF-8"
   ```
2. 进入 `JBoss/standalone/configuration/standalone.xml`
   ```xml
   <datasource jndi-name="java:/MySqlDS" pool-name="MySqlDS" enabled="true">
       <connection-url>jdbc:mysql://localhost:3306/yourdb</connection-url>
       <driver>mysql</driver>
       <security>
           <user-name>root</user-name>
           <password>yourpassword</password>
       </security>
   </datasource>
   ```

### **8.3 运行 Web 应用**
1. 右键项目 → **Run As → Run on Server**
2. 访问 `http://localhost:8080/JBossSpringMVC/`

---

## **总结**
- **Eclipse 项目编码：UTF-8**
- **JBoss 服务器**
- **Spring MVC + JSP**
- **MyBatis + MySQL**
- **静态页面 `/WEB-INF/views/*.jsp`**
- **动态页面 `/WEB-INF/web.xml` 配置**

你觉得这个方案符合你的需求吗？

## Z025/03/07（五）

如果你想要一个**可以在页面上随意拖拽移动图标（或元素）**的**开源框架**，可以考虑以下几种方案：  

---

### **1. [Interact.js](https://interactjs.io/)（推荐）**  
- **特点**：轻量级、支持拖拽（drag）、缩放（resize）、旋转（rotate）、多点触控（multi-touch）。  
- **适用场景**：交互式 UI、拖拽排序、白板应用等。  
- **使用示例**：  
  ```html
  <script src="https://cdn.jsdelivr.net/npm/interactjs"></script>
  <div id="drag-element" style="width:100px; height:100px; background:red; position:absolute;">拖我</div>
  <script>
    interact('#drag-element').draggable({
      listeners: {
        move(event) {
          event.target.style.transform = `translate(${event.pageX}px, ${event.pageY}px)`;
        }
      }
    });
  </script>
  ```

---

### **2. [Draggable (by Shopify)](https://shopify.github.io/draggable/)**  
- **特点**：Shopify 开源的拖拽库，支持**拖拽排序、分组、吸附**等功能。  
- **适用场景**：拖拽列表、卡片布局、UI 组件交互等。  
- **使用示例**：  
  ```html
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable/lib/draggable.bundle.legacy.js"></script>
  <div class="draggable-container">
    <div class="draggable">拖动我</div>
  </div>
  <script>
    const draggable = new Draggable.Draggable(document.querySelectorAll('.draggable-container'), {
      draggable: '.draggable'
    });
  </script>
  ```

---

### **3. [Sortable.js](https://sortablejs.github.io/Sortable/)**  
- **特点**：支持拖拽排序、列表拖拽交换，适用于**拖拽排序**需求。  
- **适用场景**：Todo 列表、任务管理工具、Kanban 看板等。  
- **使用示例**：  
  ```html
  <script src="https://cdn.jsdelivr.net/npm/sortablejs"></script>
  <ul id="sortable-list">
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
  </ul>
  <script>
    new Sortable(document.getElementById('sortable-list'), { animation: 150 });
  </script>
  ```

---

### **4. [Konva.js](https://konvajs.org/)（适用于 Canvas 拖拽）**  
- **特点**：如果你的图标是 **Canvas 元素**，Konva.js 提供了强大的**拖拽、缩放、层管理**功能。  
- **适用场景**：图形编辑器、白板、游戏开发等。  
- **使用示例**：  
  ```html
  <script src="https://cdn.jsdelivr.net/npm/konva@8.3.12/konva.min.js"></script>
  <div id="container"></div>
  <script>
    var stage = new Konva.Stage({ container: 'container', width: 500, height: 500 });
    var layer = new Konva.Layer();
    var circle = new Konva.Circle({ x: 100, y: 100, radius: 30, fill: 'red', draggable: true });
    layer.add(circle);
    stage.add(layer);
  </script>
  ```

---

## **总结：哪个框架适合你？**
| 框架 | 适用场景 | 特色 |
|------|---------|------|
| **Interact.js** | 任意元素拖拽 | 轻量级、功能全面 |
| **Draggable (Shopify)** | 拖拽 UI 组件 | 适合卡片、列表拖拽 |
| **Sortable.js** | 拖拽排序 | 适用于任务管理、看板 |
| **Konva.js** | 拖拽 Canvas 图形 | 适用于绘图、游戏开发 |

如果你的需求是**普通网页上的图标拖拽**，推荐 **Interact.js** 或 **Draggable.js**。如果是**排序**，用 **Sortable.js**，如果是**Canvas 图形**，用 **Konva.js**。  

你具体的需求是什么？我可以帮你选择最合适的方案！

在拖拽功能中，**分组（Grouping）** 和 **吸附（Snapping）** 是两个常见的高级功能，常用于 UI 设计、拖拽排序、白板应用等。  

---

## **1. 分组（Grouping）**
### **📌 作用：**
- 允许多个元素作为**一个整体**进行拖拽，而不是单独拖动每个元素。  
- 适用于**拖拽多个元素（如文件夹、卡片组、组件组合）**。  

### **🔹 实现方法：**
使用 `Draggable.js` 或 `Interact.js`，让多个元素归属一个组（class 选择器），然后拖拽时一起移动。  

#### **✅ 方式 1：Draggable.js 实现分组**
```html
<!DOCTYPE html>
<html lang="zh">
<head>
  <script src="https://cdn.jsdelivr.net/npm/@shopify/draggable/lib/draggable.bundle.legacy.js"></script>
  <style>
    .container { display: flex; gap: 10px; }
    .group { padding: 10px; border: 1px solid black; width: 100px; text-align: center; background: lightblue; cursor: grab; }
  </style>
</head>
<body>

  <div class="container">
    <div class="group">A1</div>
    <div class="group">A2</div>
    <div class="group">A3</div>
  </div>

  <script>
    new Draggable.Sortable(document.querySelectorAll('.container'), {
      draggable: '.group',
      mirror: { constrainDimensions: true }
    });
  </script>

</body>
</html>
```
✅ **解释：**
- `draggable: '.group'` 指定 **.group** 内的元素可以拖动。  
- `mirror: { constrainDimensions: true }` 让拖拽的元素保持原始大小，防止变形。  

---

#### **✅ 方式 2：Interact.js 实现分组**
```html
<script src="https://cdn.jsdelivr.net/npm/interactjs"></script>
<style>
  .group { width: 100px; height: 100px; background: lightblue; position: absolute; cursor: grab; }
</style>

<div id="group1" class="group" style="top: 50px; left: 50px;">组A</div>
<div id="group2" class="group" style="top: 50px; left: 160px;">组B</div>

<script>
  interact('.group').draggable({
    listeners: {
      move(event) {
        let { target } = event;
        let x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
        let y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;
        target.style.transform = `translate(${x}px, ${y}px)`;
        target.setAttribute('data-x', x);
        target.setAttribute('data-y', y);
      }
    }
  });
</script>
```
✅ **解释：**
- 任何 `.group` 类的元素都可以拖动。
- 拖动时，会移动 `transform` 的 `translate(x, y)` 值，确保平滑移动。
- `data-x` 和 `data-y` 存储位置，防止每次拖拽都从原点开始。

---

## **2. 吸附（Snapping）**
### **📌 作用：**
- 拖拽元素时，让其自动吸附到**最近的网格、边缘、其他元素**。  
- 适用于 **拖拽对齐、网格布局、对齐参考线**。  

### **🔹 实现方法：**
使用 `Interact.js` 或 `Draggable.js`，设置 **吸附点（snap points）**。

#### **✅ Interact.js 实现吸附到网格**
```html
<script src="https://cdn.jsdelivr.net/npm/interactjs"></script>
<style>
  .box { width: 100px; height: 100px; background: lightcoral; position: absolute; cursor: grab; }
</style>

<div id="box" class="box" style="top: 100px; left: 100px;">拖我</div>

<script>
  interact('#box').draggable({
    modifiers: [
      interact.modifiers.snap({
        targets: [ interact.snappers.grid({ x: 50, y: 50 }) ], // 50px 网格吸附
        range: Infinity,
        relativePoints: [{ x: 0.5, y: 0.5 }] // 吸附到中心
      })
    ],
    listeners: {
      move(event) {
        let { target } = event;
        let x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
        let y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;
        target.style.transform = `translate(${x}px, ${y}px)`;
        target.setAttribute('data-x', x);
        target.setAttribute('data-y', y);
      }
    }
  });
</script>
```
✅ **解释：**
- `snap({ targets: [ interact.snappers.grid({ x: 50, y: 50 }) ] })` → **让元素每 50px 吸附一次**，形成网格布局。  
- `relativePoints: [{ x: 0.5, y: 0.5 }]` → **吸附到元素中心**。

---

#### **✅ Interact.js 实现吸附到其他元素**
```html
<script src="https://cdn.jsdelivr.net/npm/interactjs"></script>
<style>
  .box { width: 100px; height: 100px; background: lightgreen; position: absolute; cursor: grab; }
</style>

<div id="target" class="box" style="top: 200px; left: 200px;">目标</div>
<div id="draggable" class="box" style="top: 100px; left: 100px;">拖我</div>

<script>
  interact('#draggable').draggable({
    modifiers: [
      interact.modifiers.snap({
        targets: [
          { x: 200, y: 200 }, // 吸附到 (200,200)
          { x: 250, y: 250 }  // 吸附到 (250,250)
        ],
        range: 30, // 30px 内自动吸附
        relativePoints: [{ x: 0.5, y: 0.5 }]
      })
    ],
    listeners: {
      move(event) {
        let { target } = event;
        let x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
        let y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;
        target.style.transform = `translate(${x}px, ${y}px)`;
        target.setAttribute('data-x', x);
        target.setAttribute('data-y', y);
      }
    }
  });
</script>
```
✅ **解释：**
- `targets: [{ x: 200, y: 200 }, { x: 250, y: 250 }]` → **拖动时会自动吸附到这些点**。
- `range: 30` → **如果鼠标在 30px 以内，自动吸附**。

---

## **总结**
| 功能 | 适用场景 | 实现方法 |
|------|---------|---------|
| **分组（Grouping）** | 拖拽多个元素、整体移动 | Draggable.js / Interact.js |
| **吸附到网格** | 让元素对齐网格（50px/100px 等） | Interact.js（grid snap） |
| **吸附到其他元素** | 自动吸附到指定位置（如参考线） | Interact.js（snap points） |

如果你的需求是**多元素拖拽**，推荐 **Draggable.js**；如果是**网格对齐、吸附**，推荐 **Interact.js**。  

你打算实现什么样的拖拽功能？我可以帮你优化代码！


### 主页面是随意移动的圆形图标，像气泡一样，根据日期，现在时刻早中晚，显示的图标是不一样的，当选中图标时，和它相关的图标依附过来

你的需求类似于一个**“智能气泡界面”**，核心功能包括：  
1. **气泡（圆形图标）自由拖动**。  
2. **不同时间段（早、中、晚）显示不同的图标**。  
3. **选中某个图标时，相关图标会吸附靠近**。  

---

## **技术选型**
| 功能 | 方案 | 相关库 |
|------|------|------|
| **自由拖拽** | **每个气泡可以随意拖动** | `Interact.js` / `Draggable.js` |
| **时间控制** | **根据当前时间更换图标** | `JavaScript Date()` |
| **吸附功能** | **选中一个气泡，相关气泡靠近** | `Interact.js`（snap） |
| **动态数据** | **可从 API / 本地 JSON 读取数据** | `fetch()` 或 `localStorage` |

---

## **💡 代码实现**
### **📌 1. HTML 结构**
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>气泡界面</title>
    <script src="https://cdn.jsdelivr.net/npm/interactjs"></script>
    <style>
        body { background-color: #222; overflow: hidden; }
        .bubble {
            position: absolute;
            width: 80px; height: 80px;
            border-radius: 50%;
            background-color: lightblue;
            display: flex; align-items: center; justify-content: center;
            text-align: center;
            font-size: 16px; color: black;
            cursor: grab;
            transition: transform 0.3s ease-out;
        }
    </style>
</head>
<body>

    <!-- 气泡示例 -->
    <div id="bubble1" class="bubble" data-type="morning" style="top: 100px; left: 150px;">早晨</div>
    <div id="bubble2" class="bubble" data-type="afternoon" style="top: 200px; left: 300px;">下午</div>
    <div id="bubble3" class="bubble" data-type="evening" style="top: 300px; left: 450px;">晚上</div>

    <script>
        // 1️⃣ 获取当前时间，显示不同的气泡
        function updateBubbles() {
            let hour = new Date().getHours();
            let type = hour < 12 ? "morning" : hour < 18 ? "afternoon" : "evening";

            document.querySelectorAll('.bubble').forEach(bubble => {
                bubble.style.display = bubble.dataset.type === type ? "block" : "none";
            });
        }
        updateBubbles(); // 页面加载时检查

        // 2️⃣ 让气泡可拖拽
        interact('.bubble').draggable({
            inertia: true,
            listeners: {
                move(event) {
                    let target = event.target;
                    let x = (parseFloat(target.getAttribute('data-x')) || 0) + event.dx;
                    let y = (parseFloat(target.getAttribute('data-y')) || 0) + event.dy;
                    target.style.transform = `translate(${x}px, ${y}px)`;
                    target.setAttribute('data-x', x);
                    target.setAttribute('data-y', y);
                }
            }
        });

        // 3️⃣ 选中气泡时，相关气泡吸附靠近
        document.querySelectorAll('.bubble').forEach(bubble => {
            bubble.addEventListener("click", function() {
                let selectedType = this.dataset.type;
                document.querySelectorAll('.bubble').forEach(otherBubble => {
                    if (otherBubble !== this && otherBubble.dataset.type === selectedType) {
                        let newX = parseFloat(this.getAttribute('data-x')) + 50; // 让相关气泡靠近
                        let newY = parseFloat(this.getAttribute('data-y')) + 50;
                        otherBubble.style.transform = `translate(${newX}px, ${newY}px)`;
                        otherBubble.setAttribute('data-x', newX);
                        otherBubble.setAttribute('data-y', newY);
                    }
                });
            });
        });

    </script>

</body>
</html>
```

---

## **🔹 代码解析**
1. **按时间段显示不同气泡**
   - 通过 `new Date().getHours()` 获取当前时间。
   - `早晨（0-11点）`，`下午（12-17点）`，`晚上（18-23点）` 。
   - 仅显示符合当前时间的气泡，其他隐藏。

2. **拖拽功能**
   - `Interact.js` 让每个 `.bubble` 可以拖拽。
   - `transform: translate(x, y)` 记录并更新气泡位置。

3. **吸附功能**
   - 选中一个气泡后，**同类型（同时间段）的气泡自动靠近**。
   - 通过 `setAttribute('data-x', x)` 记录位置，保证连续拖动时不会乱跳。

---

## **🎯 可扩展功能**
| 需求 | 方案 |
|------|------|
| **不同时间段的气泡不同颜色** | 可在 `updateBubbles()` 里为 `bubble.style.backgroundColor` 设置颜色 |
| **点击后相关气泡自动围绕** | 用 `Math.sin()` 和 `Math.cos()` 计算环形排列 |
| **数据从 API 读取** | 通过 `fetch()` 获取远程 JSON 数据动态创建气泡 |
| **吸附增强** | 使用 `interact.modifiers.snap()` 让气泡吸附到网格 |

---

## **🚀 总结**
- **基础功能**：支持**拖拽、自动吸附、根据时间显示不同气泡**。  
- **高级扩展**：可以**设置不同颜色、数据动态加载、吸附增强**。  
- **使用技术**：
  - `Interact.js`（拖拽 + 吸附）
  - `JavaScript Date()`（时间控制）
  - `CSS transition`（动画效果）

如果你有更详细的需求，比如**让气泡自动围绕选中气泡旋转、增加弹性效果**等，可以告诉我，我可以进一步优化代码！
