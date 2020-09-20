# Toolbar工具栏

<cite>
**本文档引用文件**  
- [index.vue](file://src/views/index.vue)
- [mixin.js](file://src/views/mixin.js)
- [g6-editor.md](file://doc/v1/g6-editor.md)
</cite>

## 目录
1. [工具栏实例化与DOM绑定](#工具栏实例化与dom绑定)
2. [命令系统映射机制](#命令系统映射机制)
3. [支持的命令及其功能](#支持的命令及其功能)
4. [自定义保存命令实现](#自定义保存命令实现)
5. [非命令按钮交互模式](#非命令按钮交互模式)
6. [图标与工具提示设计](#图标与工具提示设计)
7. [自定义工具按钮扩展方案](#自定义工具按钮扩展方案)

## 工具栏实例化与DOM绑定
在`src/views/index.vue`文件中，通过`new G6Editor.Toolbar({ container: 'toolbar' })`完成工具栏的实例化。该过程在`initG6Editor`方法中执行，首先创建G6Editor实例，然后创建Toolbar实例并指定其容器为ID为`toolbar`的DOM元素。最后通过`editor.add(toolbar)`将工具栏组件添加到编辑器实例中，完成组件的挂载和初始化。

**Section sources**
- [index.vue](file://src/views/index.vue#L274-L370)

## 命令系统映射机制
工具栏中的每个`i`标签通过`data-command`属性与G6 Editor命令系统建立映射关系。当用户点击带有`data-command`属性的按钮时，G6 Editor会自动识别该属性值并执行对应的命令。这种机制要求按钮必须同时具有`command`类和`data-command`属性，这是G6 Editor的规范要求。命令的执行流程由G6 Editor内部的命令调度系统管理，确保点击操作能够准确触发相应的功能逻辑。

**Section sources**
- [g6-editor.md](file://doc/v1/g6-editor.md#L48-L61)
- [index.vue](file://src/views/index.vue#L1-L50)

## 支持的命令及其功能
工具栏支持以下命令及其功能：
- **保存**：`data-command='save'`，保存当前画布数据
- **撤销**：`data-command='undo'`，撤销上一步操作
- **重做**：`data-command='redo'`，重做被撤销的操作
- **删除**：`data-command='delete'`，删除选中元素
- **缩小**：`data-command='zoomOut'`，缩小画布视图
- **放大**：`data-command='zoomIn'`，放大画布视图
- **清除画布**：`data-command='clear'`，清空画布所有元素
- **提升层级**：`data-command='toFront'`，将选中元素置于顶层
- **下降层级**：`data-command='toBack'`，将选中元素置于底层
- **全选**：`data-command='selectAll'`，选中画布上所有元素
- **复制**：`data-command='copy'`，复制选中元素
- **粘贴**：`data-command='paste'`，粘贴已复制的元素
- **实际大小**：`data-command='autoZoom'`，将画布缩放至实际大小
- **适应页面**：`data-command='resetZoom'`，调整画布以适应页面
- **组合**：`data-command='addGroup'`，将多个元素组合为一个组
- **取消组合**：`data-command='unGroup'`，将组合的元素拆分为独立元素
- **多选**：`data-command='multiSelect'`，启用多选模式

**Section sources**
- [index.vue](file://src/views/index.vue#L1-L50)
- [g6-editor.md](file://doc/v1/g6-editor.md#L810-L850)

## 自定义保存命令实现
`data-command='save'`命令通过`Command.registerCommand`在代码中自定义实现。该命令的特殊性在于其包含localStorage持久化和控制台输出等业务逻辑。在`initG6Editor`方法中，通过`Command.registerCommand("save", {...})`注册自定义命令，其`execute`方法中执行`editor.getCurrentPage().save()`获取当前画布数据，使用`localStorage.setItem("flowData", JSON.stringify(needSaveData))`进行持久化存储，并通过`_this.$message.success("数据已保存")`显示成功提示。该命令被设置为`queue: false`，表示不进入命令队列，因此不会影响撤销/重做功能。

**Section sources**
- [index.vue](file://src/views/index.vue#L274-L324)
- [g6-editor.md](file://doc/v1/g6-editor.md#L852-L919)

## 非命令按钮交互模式
非`data-command`按钮通过`@click`绑定Vue方法实现交互模式。这些按钮包括"历史数据"、"上传数据"、"另存为文件"和"另存为图片"。它们不参与G6 Editor的命令系统，而是直接绑定到Vue组件的方法上：
- `@click="readHistoryData"`：读取localStorage中的历史数据
- `@click="readUploadData"`：通过文件输入控件上传JSON数据
- `@click="saveAsFile"`：将当前画布数据保存为JSON文件
- `@click="openSaveAsImageDialog"`：打开保存为图片的对话框
这种模式提供了更灵活的业务逻辑处理能力，适用于需要复杂数据处理或文件操作的场景。

**Section sources**
- [index.vue](file://src/views/index.vue#L1-L50)
- [index.vue](file://src/views/index.vue#L434-L466)

## 图标与工具提示设计
工具栏图标使用font-awesome集成，通过在`i`标签上添加`fa`和具体图标类名（如`fa-floppy-o`）来显示相应图标。这种集成方式在`src/main.js`中通过`import "font-awesome/css/font-awesome.min.css"`引入。工具提示通过`title`属性实现，当用户将鼠标悬停在按钮上时，浏览器会自动显示`title`属性的值作为提示信息。这种设计提升了用户体验，使用户能够直观地理解每个按钮的功能，而无需记忆图标含义。

**Section sources**
- [main.js](file://src/main.js#L1-L17)
- [index.vue](file://src/views/index.vue#L1-L50)

## 自定义工具按钮扩展方案
扩展自定义工具按钮有两种实现路径：
1. **HTML标记添加**：在工具栏的`div#toolbar`中添加新的`i`标签，设置`data-command`属性和`command`类，然后通过`Command.registerCommand`注册对应命令
2. **事件绑定**：添加不带`data-command`属性的按钮，通过`@click`直接绑定Vue方法，实现特定功能
第一种方式适用于需要集成到G6 Editor命令系统中的功能，支持快捷键和命令队列；第二种方式适用于独立的业务功能，提供更大的灵活性。无论哪种方式，都应确保按钮样式与现有工具栏一致，并提供清晰的工具提示。

**Section sources**
- [index.vue](file://src/views/index.vue#L1-L50)
- [g6-editor.md](file://doc/v1/g6-editor.md#L852-L919)