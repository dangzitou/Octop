# Agent Mail #1148 验收

环境：macOS arm64，官方 CLI 1.0.18，真实模型 step-5-preview；使用独立临时 Octop 实例和两个真实已授权邮箱。

## 过程与结果

1. 在 Octop 创建 Agent Mail 实例，检测 CLI，完成人类设备授权；刷新后仍为已授权。
2. 向用户控制的测试收件箱发送、回复、转发，共 3 封，均取得自动回执；用户手工回信和 TXT 附件已收到。
3. 原始、转发、人工回信附件均为 76 字节，逐字节一致。SHA-256：`0b36569400e8c633ced39e21fd33242e03c3dea71a0213cb97259fcd60a8e066`。
4. 五种写工具均先预览。只确认清理核对过的测试副本，修改目标和重放批准被拒绝。
5. Step Agent 实际调用身份、搜索、读取、下载工具；附件大小和哈希独立核对一致。两轮“准备发信 → 取消”工具调用数为 `[1, 0]`，发件箱未变化。
6. A、B 两个真实邮箱的 Step Agent 并发检查身份和搜索，各自只调用所选实例的工具；A 的邮件在 B 中不存在。刷新 B 后两者身份不变。B 直接读取 A 的邮件 ID 被拒绝，官方 CLI 返回 `404 / Resource not found`，A 仍可读取。

模型首次触及测试 Agent 的 24 次迭代上限，最终设置 100 次后完成。模型测试覆盖只读和预览后取消；确认执行由真实邮箱 MCP 测试覆盖。

7. 在任务页为 A、B 两个邮箱配置新邮件触发任务，模型选 `step-5-preview`。监听空闲 20 秒，两个任务均未调用模型。
8. 两个已授权测试邮箱互发各一封编号邮件；官方 `message +watch` 分别触发一次任务，Step 实际调用对应邮箱的读取工具并报告正确主题和正文。邮件中要求输出 `WRONG_RESULT` 的测试指令被忽略。执行日志确认两次成功，模型使用记录为 `step-5-preview`。
9. 在界面停用并编辑保存 A，核对邮箱触发值和模型未变化；停用 B 后，监听进程由 2 个降到 0 个。
10. 最终补充监听与任务取消边界后，用上述真实邮件 ID 重放两个事件，再由真实 Step 模型各读取一次；未额外发信。自动回归覆盖任务自停、进程回收、实例隔离及事件去重。

最终检查：`make all PYTEST_JOBS=4` 为 **4212 passed / 17 skipped**；Windows 平台 mypy、前端类型检查、修改文件 ESLint、相关前端 23 个测试及生产构建通过。

## 实际使用界面

### Octop 连接器已授权

![连接器已授权](01-connector-authorized.jpg)

### 两个连接器及第二个邮箱授权

![第二个邮箱已授权](03-second-mailbox-authorized.jpg)

### Step 模型实际读取人工回信附件

![实际 Step 模型会话](04-real-step-read.jpg)

结构化断言：[acceptance.json](acceptance.json)。

### 新邮件事件任务使用 Step 模型，执行成功

![新邮件触发任务](05-watch-task-step.jpg)

### 实际收到邮件后，Step 读取并返回验收字段

![Step 处理新邮件](06-watch-real-step-read.jpg)

新邮件推送断言：[watch-acceptance.json](watch-acceptance.json)。
