# 工作状态保存功能说明

## 新增功能概述

treeview 现在支持保存和恢复工作状态，包括：
- ✅ 目录的折叠/展开状态（col）
- ✅ 目录/文件的隐藏状态（hidden）
- ✅ 目录/文件的删除状态（deleted）
- ✅ 添加的注释（comment）
- ✅ 时间戳（timestamp）

## 使用流程

### 第一次使用（保存状态）

1. 打开 tree 文件：
```bash
tree /path/to/project | ./treeview
```

2. 在浏览器中整理树结构：
   - 点击三角形展开/折叠目录
   - 点击「+ 注释」为目录添加说明
   - 点击「隐藏」隐藏不需要的项目
   - 点击「删除」移除项目

3. 整理完成后，点击「💾 保存状态」按钮
   - 会自动下载 `treeview-state.json` 文件
   - 建议将此文件放入项目

### 后续使用（恢复状态）

**方式 A：命令行加载（推荐开发流程中使用）**

```bash
tree /path/to/project | ./treeview -s treeview-state.json
```

打开浏览器时，会自动应用之前保存的所有状态。

**方式 B：浏览器手动导入**

1. 打开 tree 文件
2. 内容加载完成后，点击「📥 导入状态」按钮
3. 选择之前保存的 `treeview-state.json` 文件
4. 页面会立即恢复之前的所有整理状态

## 状态文件结构

```json
{
  "nodes": {
    "path/to/dir": {
      "comment": "这是一个注释",
      "col": false,           // false=展开, true=折叠
      "hidden": false         // false=可见, true=隐藏
    },
    "another/path": {
      "comment": "",
      "col": true,
      "hidden": false
    }
  },
  "deletedPaths": [
    "path/to/deleted/item1",
    "path/to/deleted/item2"
  ],
  "timestamp": "2026-05-22T12:34:56.789Z"
}
```

## 工作流示例

### 场景：整理项目文档示例

```bash
# 第一次：生成 tree 并整理
tree -L 3 ./my-project | ./treeview

# 在浏览器中：
# - 展开 src, docs
# - 折叠测试和构建目录
# - 给关键目录添加注释
# - 删除不相关目录
# - 点击「💾 保存状态」

# 保存的 treeview-state.json 提交到仓库
git add treeview-state.json
git commit -m "save treeview state"
```

```bash
# 后续：快速恢复整理状态
tree -L 3 ./my-project | ./treeview -s treeview-state.json

# 或在页面中导入：
# 1. tree -L 3 ./my-project | ./treeview
# 2. 点击「📥 导入状态」
# 3. 选择 treeview-state.json
```

## 技术细节

### 前端（treeview.html）

新增函数：
- `saveState()` - 收集当前状态并下载 JSON
- `loadState(state)` - 加载 JSON 状态文件
- `collectNodeState(node, states, deletedPaths)` - 递归收集节点状态
- `applyNodeState(node, states)` - 应用状态到节点
- `setNodeDeleted(node, deletedPaths)` - 标记删除的节点
- `onStateFile(e)` - 处理状态文件上传

修改的地方：
- 删除操作现在标记 `node.deleted = true` 而不是完全移除节点
- `mkNode()` 跳过已删除的节点
- `render()` 自动尝试从 `/state` 端点加载状态

### 后端（treeview）

新增功能：
- 命令行参数解析：`-s` 或 `--state <file>`
- 新的 `/state` API 端点返回保存的状态 JSON
- 状态文件在启动时加载

## 常见问题

**Q: 什么时候选用命令行加载 vs 浏览器导入？**
A: 命令行加载更快，适合频繁使用同一个 tree 的场景（比如翻译或举例）。浏览器导入更灵活，适合临时加载不同的状态文件。

**Q: 删除后的内容还能恢复吗？**
A: 可以！从状态文件中删除 `deletedPaths` 数组中的对应路径，然后重新导入即可。

**Q: 更新 tree 后，之前的状态还能用吗？**
A: 部分可以。状态基于节点路径匹配：
- 新增节点会以默认状态显示
- 已删除的节点不会影响
- 其他节点的状态会正确保留

**Q: 可以手动编辑 JSON 文件吗？**
A: 可以！JSON 格式很简单，可以用文本编辑器修改注释、折叠状态等。

## 示例状态文件

已提供示例文件：
- `test_tree.txt` - 示例 tree 输出
- `test_state.json` - 示例状态文件

快速测试：
```bash
cat test_tree.txt | ./treeview -s test_state.json
```
