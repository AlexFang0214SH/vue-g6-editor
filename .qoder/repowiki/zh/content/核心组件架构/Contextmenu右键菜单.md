# Contextmenu右键菜单

<cite>
**本文档引用的文件**
- [index.vue](file://src/views/index.vue)
- [index.less](file://src/views/index.less)
- [g6-editor.md](file://doc/v1/g6-editor.md)
</cite>

## 目录
1. [简介](#简介)
2. [初始化与DOM绑定](#初始化与dom绑定)
3. [上下文敏感菜单状态匹配](#上下文敏感菜单状态匹配)
4. [各状态下可用命令按钮](#各状态下可用命令按钮)
5. [data-command属性与命令系统映射](#data-command属性与命令系统映射)
6. [DOM结构与CSS样式设计](#dom结构与css样式设计)
7. [用户操作效率提升机制](#用户操作效率提升机制)
8. [自定义右键菜单项添加流程](#自定义右键菜单项添加流程)

## 简介
右键菜单（Contextmenu）是vue-g6-editor图形编辑器中的核心交互组件之一，通过上下文敏感的方式为用户提供快捷操作入口。该组件基于AntV G6Editor框架构建，利用`data-status`属性与编辑器选择状态精确匹配，动态显示对应的操作选项。菜单通过`data-command`属性直接绑定G6Editor内置命令系统，实现点击即执行的功能。其设计兼顾功能性与用户体验，显著提升了图形编辑的操作效率。

## 初始化与DOM绑定
右键菜单组件通过`new G6Editor.Contextmenu({ container: 'contextmenu' })`进行初始化，并绑定到指定的DOM容器。该过程在`src/views/index.vue`文件的`mounted`生命周期钩子中完成，具体位于`initG6Editor`方法内。

```javascript
const contextmenu = new G6Editor.Contextmenu({
  container: "contextmenu"
});
editor.add(contextmenu);
```

此初始化过程将右键菜单实例注册到G6Editor主编辑器中，使其能够监听编辑器状态变化并响应用户右键操作。`container`配置项指定为`contextmenu`，对应HTML中id为`contextmenu`的div元素，完成组件与DOM的绑定。

**Section sources**
- [index.vue](file://src/views/index.vue#L274-L324)

## 上下文敏感菜单状态匹配
右键菜单通过`data-status`属性与G6 Editor的选择状态实现精确匹配，确保仅在合适的情境下显示相关操作。菜单包含五个独立区域，分别对应不同的选择状态：

- `data-status="node-selected"`：节点选中状态
- `data-status="edge-selected"`：边选中状态
- `data-status="group-selected"`：群组选中状态
- `data-status="canvas-selected"`：画布选中状态
- `data-status="multi-selected"`：多选状态

G6Editor框架通过`statuschange`事件监听状态变化，并自动控制对应`data-status`区域的显示与隐藏。这种机制确保了菜单内容始终与当前编辑情境保持一致，避免了无关操作项的干扰。

**Section sources**
- [index.vue](file://src/views/index.vue#L206-L233)
- [g6-editor.md](file://doc/v1/g6-editor.md#L286-L319)

## 各状态下可用命令按钮
不同选择状态下，右键菜单提供针对性的操作命令，具体如下：

### 节点状态（node-selected）
- **复制**：`data-command="copy"` - 复制选中节点
- **粘贴**：`data-command="paste"` - 粘贴已复制的节点
- **删除**：`data-command="delete"` - 删除选中节点

### 边状态（edge-selected）
- **删除**：`data-command="delete"` - 删除选中边

### 群组状态（group-selected）
- **复制**：`data-command="copy"` - 复制选中群组
- **粘贴**：`data-command="paste"` - 粘贴已复制的群组
- **取消组合**：`data-command="unGroup"` - 解散当前群组
- **删除**：`data-command="delete"` - 删除选中群组

### 画布状态（canvas-selected）
- **撤销**：`data-command="undo"` - 撤销上一步操作
- **重做**：`data-command="redo"` - 重做被撤销的操作（当前禁用）

### 多选状态（multi-selected）
- **复制**：`data-command="copy"` - 复制所有选中元素
- **粘贴**：`data-command="paste"` - 粘贴已复制的元素
- **组合**：`data-command="addGroup"` - 将选中元素组合为群组

**Section sources**
- [index.vue](file://src/views/index.vue#L206-L233)

## data-command属性与命令系统映射
每个`el-button`元素的`data-command`属性直接映射到G6 Editor的内置命令系统，实现点击即执行的功能。当用户点击带有`data-command`属性的按钮时，G6Editor会自动触发对应的命令执行流程。

例如：
- `data-command="copy"` 触发复制命令
- `data-command="delete"` 触发删除命令
- `data-command="undo"` 触发撤销命令

这些命令由G6Editor.Command系统管理，支持快捷键绑定、队列管理（用于撤销/重做）和启用状态控制。自定义命令（如`save`）也可通过`Command.registerCommand()`方法注册，并与`data-command`属性关联。

**Section sources**
- [index.vue](file://src/views/index.vue#L274-L324)
- [g6-editor.md](file://doc/v1/g6-editor.md#L852-L919)

## DOM结构与CSS样式设计
右键菜单采用清晰的DOM结构设计，通过CSS实现浮动显示效果。

### DOM结构
```html
<div id="contextmenu">
  <div data-status="node-selected" class="menu">
    <el-button data-command="copy" class="command">复制</el-button>
    <!-- 其他按钮 -->
  </div>
  <!-- 其他状态区域 -->
</div>
```

- 外层`div#contextmenu`作为容器
- 每个`data-status`区域用`div.menu`包裹按钮组
- 使用`el-button`组件实现按钮，保留`command`类名以确保命令绑定

### CSS样式
在`src/views/index.less`中，通过以下样式控制菜单外观：
- `#contextmenu { display: none; }`：初始隐藏菜单，避免默认显示
- `.menu /deep/ .el-button`：设置按钮样式，包括宽度、边框和布局
- `:nth-last-of-type(1)`：为最后一个按钮添加底部边框

这种设计确保菜单在右键点击时由G6Editor控制显示，并通过绝对定位实现浮动效果。

**Section sources**
- [index.vue](file://src/views/index.vue#L206-L233)
- [index.less](file://src/views/index.less#L130-L146)

## 用户操作效率提升机制
右键菜单通过以下方式显著增强用户操作效率：
- **情境化操作**：根据当前选择对象自动显示相关命令，减少无关选项干扰
- **快捷访问**：无需切换工具栏即可执行常用操作，缩短操作路径
- **一致性体验**：与工具栏使用相同的`data-command`系统，保持操作逻辑统一
- **视觉反馈**：通过禁用状态（如"重做"按钮的`disable`类）清晰传达当前可执行操作

这种设计遵循了"就近原则"，将高频操作直接暴露在用户操作点附近，极大提升了图形编辑的工作流效率。

**Section sources**
- [index.vue](file://src/views/index.vue#L206-L233)

## 自定义右键菜单项添加流程
添加自定义右键菜单项的完整流程如下：

### 1. 扩展HTML结构
在`div#contextmenu`中添加新的菜单项，保持`data-status`和`class="command"`规范：
```html
<div data-status="node-selected" class="menu">
  <el-button data-command="customAction" class="command">自定义操作</el-button>
</div>
```

### 2. 绑定data-command属性
为新按钮设置唯一的`data-command`值，如`customAction`。

### 3. 注册对应命令
在`initG6Editor`方法中使用`Command.registerCommand()`注册新命令：
```javascript
Command.registerCommand("customAction", {
  enable: (editor) => true,
  execute: (editor) => {
    // 自定义逻辑
  }
});
```

### 4. （可选）启用快捷键
若需快捷键支持，在`Flow`配置中开启：
```javascript
const flow = new G6Editor.Flow({
  shortcut: {
    customAction: true
  }
});
```

此流程确保了自定义菜单项与G6Editor命令系统的无缝集成。

**Section sources**
- [index.vue](file://src/views/index.vue#L274-L324)
- [g6-editor.md](file://doc/v1/g6-editor.md#L852-L919)