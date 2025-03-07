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

