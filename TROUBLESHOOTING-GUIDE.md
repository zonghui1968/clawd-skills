# 🚨 游戏故障排查指南

## 问题：两款游戏都不能正常运行

我已经为你创建了一个**自动化诊断工具**来检测问题！

---

## 📋 第一步：运行诊断工具

### 打开诊断页面
```
文件位置: C:\Users\宗晖\clawd\tetris-game\diagnose.html
```

### 操作方法
1. 在文件资源管理器中找到 `diagnose.html`
2. 双击打开（会在浏览器中打开）
3. 点击两个游戏的"运行测试"按钮
4. 查看测试结果

诊断工具会自动检查：
- ✅ 文件是否存在
- ✅ 浏览器兼容性
- ✅ Canvas 支持
- ✅ localStorage 支持
- ✅ 游戏逻辑完整性
- ✅ 服务器连接状态

---

## 🧩 俄罗斯方块 - 常见问题

### 问题1：双击 index.html 没有反应

**可能原因：**
- 文件关联问题
- 浏览器未设置为默认浏览器

**解决方法：**
1. 右键点击 `index.html`
2. 选择"打开方式" → 选择浏览器（Chrome/Edge）
3. 或使用诊断工具中的"打开游戏"按钮

### 问题2：游戏页面空白

**可能原因：**
- JavaScript 文件路径错误
- 浏览器控制台有错误

**解决方法：**
1. 按 F12 打开开发者工具
2. 查看 Console 标签页是否有错误
3. 检查 Network 标签，看 `tetris.js` 是否加载成功

**手动检查：**
```html
<!-- 确保 index.html 中有这一行 -->
<script src="tetris.js"></script>
```

### 问题3：按空格键游戏不开始

**可能原因：**
- JavaScript 未正确初始化
- 事件监听器未绑定

**检查方法：**
1. 打开浏览器控制台（F12）
2. 输入：`window.tetris`
3. 应该看到 TetrisGame 对象，如果显示 undefined，说明游戏未初始化

**临时解决：**
在控制台输入：
```javascript
location.reload();
```

### 问题4：方块不移动

**可能原因：**
- Canvas 未正确渲染
- 键盘事件未捕获

**检查方法：**
1. 点击游戏区域确保获得焦点
2. 查看浏览器控制台是否有错误
3. 尝试点击页面后再按方向键

---

## 🎨 你画我猜 - 常见问题

### 问题1：无法访问 localhost:3001

**错误信息：**
- "无法访问此网站"
- "localhost 拒绝了连接"

**原因：服务器未启动**

**解决方法：**
```bash
# 打开 PowerShell 或 CMD
cd C:\Users\宗晖\clawd\draw-and-guess
npm start
```

**看到这个消息说明成功：**
```
Draw & Guess game server running on port 3001
Open http://localhost:3001 in your browser
```

### 问题2：端口 3001 被占用

**错误信息：**
```
Error: listen EADDRINUSE: address already in use :::3001
```

**解决方法1：杀掉占用端口的进程**
```bash
# 查找占用端口的进程
netstat -ano | findstr :3001

# 终止进程（替换 PID 为实际进程 ID）
taskkill /PID <PID> /F

# 重新启动服务器
cd C:\Users\宗晖\clawd\draw-and-guess
npm start
```

**解决方法2：更换端口**
编辑 `src/server.js`：
```javascript
// 将这一行
const PORT = process.env.PORT || 3001;
// 改为其他端口，比如 3002
const PORT = process.env.PORT || 3002;
```

然后访问新的端口：http://localhost:3002

### 问题3：页面打开但无法连接

**可能原因：**
- WebSocket 连接失败
- 客户端 JavaScript 错误

**检查方法：**
1. 按 F12 打开开发者工具
2. 查看 Console 标签页
3. 应该看到 "Connected to server"

**如果看到连接错误：**
```
WebSocket connection to 'ws://localhost:3001/' failed
```

**解决方法：**
1. 确认服务器正在运行
2. 检查防火墙是否阻止了连接
3. 尝试使用其他浏览器

### 问题4：无法创建或加入房间

**可能原因：**
- WebSocket 消息传递失败
- 输入验证问题

**检查方法：**
1. 打开浏览器控制台（F12）
2. 点击"创建房间"或"加入房间"
3. 查看控制台是否有错误

**常见错误：**
- "Please enter your name" → 需要先输入名字
- "Invalid room code" → 房间号格式错误（应为6位大写字母）

### 问题5：绘画不同步

**可能原因：**
- WebSocket 数据传输问题
- Canvas 事件处理错误

**检查方法：**
1. 打开两个浏览器窗口
2. 创建并加入同一个房间
3. 在一个窗口绘画
4. 查看另一个窗口是否实时显示

**如果不同步：**
1. 刷新两个页面
2. 重新创建和加入房间
3. 检查浏览器控制台的错误信息

---

## 🔍 通用调试步骤

### 步骤1：清除浏览器缓存
```
Chrome: Ctrl + Shift + Delete
Edge: Ctrl + Shift + Delete
Firefox: Ctrl + Shift + Delete
```

### 步骤2：禁用浏览器扩展
某些扩展（广告拦截器、脚本拦截器）可能影响游戏运行

### 步骤3：尝试其他浏览器
- Chrome
- Firefox
- Edge

### 步骤4：检查文件完整性
```bash
# 俄罗斯方块
dir C:\Users\宗晖\clawd\tetris-game\public

# 应该看到：
# index.html
# styles.css
# tetris.js

# 你画我猜
dir C:\Users\宗晖\clawd\draw-and-guess\public

# 应该看到：
# index.html
# styles.css
# client.js
```

### 步骤5：查看浏览器控制台
1. 按 F12 打开开发者工具
2. 点击 Console 标签页
3. 红色错误 = 严重问题
4. 黄色警告 = 可能的问题

---

## 📞 获取帮助

如果上述方法都无法解决问题，请提供：

### 必要信息
1. **哪个游戏有问题？**
   - [ ] 俄罗斯方块
   - [ ] 你画我猜
   - [ ] 两个都有问题

2. **具体的错误信息**
   - 浏览器显示的错误
   - 控制台的错误（F12 → Console）

3. **操作步骤**
   - 你做了什么操作
   - 预期结果是什么
   - 实际结果是什么

4. **环境信息**
   - 操作系统版本
   - 浏览器类型和版本
   - Node.js 版本（对于你画我猜）

### 截图建议
- 游戏页面的截图
- 浏览器控制台的截图
- 错误消息的截图

---

## ✅ 快速检查清单

### 俄罗斯方块
- [ ] 能否打开 index.html？
- [ ] 页面是否正常显示（不是空白）？
- [ ] 浏览器控制台是否有错误？
- [ ] 按空格键是否有反应？
- [ ] 能否看到游戏元素（方块、网格）？

### 你画我猜
- [ ] 服务器是否正在运行？
- [ ] 能否访问 http://localhost:3001 ？
- [ ] 控制台是否显示 "Connected to server"？
- [ ] 能否输入名字？
- [ ] 能否创建或加入房间？

---

## 🎯 预期结果

### 俄罗斯方块
打开页面后应该看到：
- 游戏标题 "🎮 俄罗斯方块"
- 左侧：分数、等级、行数
- 中间：游戏画布（10×20 网格）
- 右侧：下一个方块预览、控制说明、最高分
- 提示："按 SPACE 开始游戏"

### 你画我猜
打开页面后应该看到：
- 游戏标题 "🎨 你画我猜"
- 名字输入框
- "创建房间" 和 "加入房间" 按钮
- 浏览器控制台显示 "Connected to server"

---

## 💡 临时解决方案

### 如果俄罗斯方块还是不能玩
尝试在线版本：
- https://tetris.com/
- https://tetris.fandom.com/

### 如果你想测试你画我猜的功能
需要：
1. 确保服务器运行
2. 打开两个浏览器窗口
3. 一个创建房间，另一个加入

---

## 📝 下一步

1. **首先**：运行诊断工具 `diagnose.html`
2. **然后**：根据测试结果排查问题
3. **如果还是不行**：告诉我具体的错误信息

我会根据你的反馈提供针对性的解决方案！🚀
