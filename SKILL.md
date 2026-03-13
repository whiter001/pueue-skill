---
name: manage-background-tasks
description: 当用户想要在后台运行命令、管理任务队列、查看任务状态或控制任务执行时使用此skill。Pueue是一个任务队列管理工具，允许用户在后台异步执行命令。此skill提供与pueue交互的所有功能，包括添加任务、查看状态、暂停/恢复任务、查看日志等。
---

# 管理后台任务 (Pueue)

## 目的

当用户想要在后台运行命令、管理任务队列、查看任务状态或控制任务执行时，使用此skill。Pueue 是一个命令行任务队列管理工具，允许用户：

- 在后台异步执行命令
- 查看和管理任务队列
- 暂停、恢复、重新启动任务
- 查看任务日志

## 前提条件

用户需要安装并运行 `pueue` 守护进程：

### 方式一：从 GitHub Releases 下载（推荐）

从 [pueue releases](https://github.com/Nukesor/pueue/releases) 页面下载对应平台的二进制文件：

```bash
# macOS (Apple Silicon)
curl -L -o /usr/local/bin/pueue https://github.com/Nukesor/pueue/releases/latest/download/pueue-aarch64-apple-darwin.tar.gz
tar -xzf pueue-aarch64-apple-darwin.tar.gz
mv pueue /usr/local/bin/
chmod +x /usr/local/bin/pueue

# macOS (Intel)
curl -L -o /usr/local/bin/pueue https://github.com/Nukesor/pueue/releases/latest/download/pueue-x86_64-apple-darwin.tar.gz

# Linux (x86_64)
curl -L -o /usr/local/bin/pueue https://github.com/Nukesor/pueue/releases/latest/download/pueue-x86_64-unknown-linux-gnu.tar.gz

# 启动守护进程
pueued -d
```

### 方式二：使用 cargo 安装

```bash
# 安装 pueue
cargo install pueue

# 启动守护进程
pueued -d
```

## 使用场景

### 1. 在后台运行命令

当用户想要：
- "在后台运行 `npm install`"
- "后台执行编译命令"
- "异步运行测试"

执行命令：

```bash
# 基本用法
pueue add "npm install"

# 带标签
pueue add -l "install" "npm install"

# 带分组
pueue add -g node "npm install"

# 延迟执行 (30分钟后)
pueue add -d "30 minutes" "make build"

# 指定工作目录
pueue add -w /path/to/project "./script.sh"

# 立即开始
pueue add -i "npm run dev"

# 立即开始并使用命令分隔符 (-- 防止命令中的参数与pueue参数冲突)
pueue add -i -- ls

# 创建为暂存状态
pueue add -s "npm install"

# 转义特殊字符
pueue add -e "echo hello world"

# 等待其他任务完成后执行
pueue add -a 1 2 3 "npm run build"

# 优先级 (数字越大越优先)
pueue add -o 10 "important-task"

# 仅返回任务ID
pueue add -p "npm install"
```

**关于命令分隔符 `--`**：

`--` 是命令行标准分隔符，用于告知 `pueue` 解析器：后面的内容是实际要执行的命令，而不是 `pueue add` 的选项。

| 写法 | 说明 |
|------|------|
| `pueue add "ls -la"` | `"ls -la"` 作为字符串整体传递 |
| `pueue add -- ls -la` | `ls -la` 被解析为命令 `ls` 加参数 `-la` |

**使用场景**：当命令本身包含可能与 `pueue` 选项混淆的参数时，必须加 `--`。

```bash
# 必须加 --，否则 --force 会被误认为 pueue 选项
pueue add -- git pull --force

# 命令参数不含 - 开头的字符时，可以省略 --
pueue add "echo hello"
```

**支持的延迟格式**：
- `18:00` - 今天18:00
- `3h` - 3小时后
- `10 minutes` - 10分钟后
- `tomorrow` - 明天
- `monday` - 下周一
- `2024-12-31T23:59:59` - 具体时间
- `4/1` - 美国日期格式
- `wednesday 10:30pm` - 具体星期几

### 2. 查看任务状态

当用户想要：
- "查看当前任务状态"
- "有哪些任务在运行"
- "显示任务队列"

执行命令：

```bash
# 查看所有任务
pueue status

# 仅显示特定分组
pueue status -g node

# JSON 格式输出
pueue status --json
```

### 3. 查看任务日志

当用户想要：
- "查看任务1的输出"
- "显示最近的日志"
- "查看编译错误"

执行命令：

```bash
# 查看任务日志
pueue log 1

# 查看多个任务日志
pueue log 1 2 3

# 查看分组日志
pueue log -g node

# 查看所有日志
pueue log -a

# 显示最后50行
pueue log -l 50

# 显示完整输出
pueue log -f

# JSON 格式
pueue log --json
```

### 4. 跟随任务输出

当用户想要：
- "实时查看任务输出"
- "像 tail -f 一样查看"

执行命令：

```bash
# 跟随任务输出
pueue follow 1

# 先显示最后10行再跟随
pueue follow -l 10 1
```

### 5. 等待任务完成

当用户想要：
- "等待任务完成后继续"
- "阻塞直到任务结束"

执行命令：

```bash
# 等待特定任务
pueue wait 1 2 3

# 等待分组所有任务
pueue wait -g node

# 等待所有任务
pueue wait -a

# 不显示日志
pueue wait -q

# 等待到特定状态 (queued, running, paused, success, failed, killed)
pueue wait -s success 1
```

### 6. 暂停任务

当用户想要：
- "暂停任务"
- "暂停所有任务"

执行命令：

```bash
# 暂停指定任务
pueue pause 1 2 3

# 暂停分组
pueue pause -g node

# 暂停所有
pueue pause -a

# 等待运行中任务完成后暂停
pueue pause -w
```

### 7. 恢复/开始任务

当用户想要：
- "恢复任务执行"
- "强制开始排队中的任务"

执行命令：

```bash
# 开始特定任务
pueue start 1 2 3

# 开始分组
pueue start -g node

# 开始所有
pueue start -a
```

### 8. 终止任务

当用户想要：
- "停止任务"
- "杀死运行中的进程"

执行命令：

```bash
# 终止指定任务
pueue kill 1 2 3

# 终止分组
pueue kill -g node

# 终止所有
pueue kill -a

# 发送特定信号
pueue kill -s SIGINT 1
pueue kill -s 9 1
```

### 9. 重启任务

当用户想要：
- "重新运行失败的任务"
- "重新执行命令"

执行命令：

```bash
# 重启特定任务
pueue restart 1 2 3

# 重启所有失败任务
pueue restart --all-failed

# 重启分组中的失败任务
pueue restart --failed-in-group node

# 立即开始
pueue restart -k 1

# 设为暂存状态
pueue restart -s 1

# 原地重启 (保留任务ID)
pueue restart -i 1

# 创建新任务重启
pueue restart --not-in-place 1

# 重启前编辑
pueue restart -e 1
```

### 10. 暂存任务

当用户想要：
- "暂停排队中的任务"
- "稍后手动开始"

执行命令：

```bash
# 暂存指定任务
pueue stash 1 2 3

# 暂存分组
pueue stash -g node

# 暂存所有
pueue stash -a

# 延迟入队
pueue stash -d "1 hour" 1
```

### 11. 入队任务

当用户想要：
- "将暂存的任务加入队列"
- "设置任务延迟执行"

执行命令：

```bash
# 入队指定任务
pueue enqueue 1 2 3

# 入队分组
pueue enqueue -g node

# 入队所有
pueue enqueue -a

# 延迟入队
pueue enqueue -d "30 minutes" 1
```

### 12. 清理完成任务

当用户想要：
- "清理已完成的任务"
- "删除成功完成的任务"

执行命令：

```bash
# 清理所有完成任务
pueue clean

# 仅清理成功任务
pueue clean -s

# 清理分组
pueue clean -g node
```

### 13. 重置队列

当用户想要：
- "清空所有任务"
- "重新开始"

执行命令：

```bash
# 重置所有
pueue reset

# 重置特定分组
pueue reset -g node build

# 强制重置
pueue reset -f
```

### 14. 移除任务

当用户想要：
- "从队列中删除任务"

执行命令：

```bash
# 移除任务 (无法移除运行中的任务，需先kill)
pueue remove 1 2 3
```

### 15. 交换任务位置

当用户想要：
- "调整任务顺序"

执行命令：

```bash
# 交换任务1和任务2的位置
pueue switch 1 2
```

### 16. 发送输入到任务

当用户想要：
- "向运行中的任务发送输入"
- "确认提示输入 y"

执行命令：

```bash
# 发送输入
pueue send 1 "y\n"

# 发送空行
pueue send 1 ""
```

### 17. 编辑任务

当用户想要：
- "修改任务属性"

执行命令：

```bash
# 编辑任务 (会打开编辑器)
pueue edit 1 2 3
```

### 18. 管理分组

当用户想要：
- "创建新的任务分组"
- "设置分组并行数"

执行命令：

```bash
# 添加分组
pueue group add build

# 添加分组并设置并行数
pueue group add -p 4 build

# 删除分组
pueue group remove build

# 设置并行数 (全局)
pueue parallel 4

# 设置分组并行数
pueue parallel -g build 2
```

## 常用命令示例

1. **后台运行 npm install 并添加标签**：
   ```bash
   pueue add -l "npm-install" "npm install"
   ```

2. **延迟30分钟后执行**：
   ```bash
   pueue add -d "30 minutes" "make build"
   ```

3. **在指定目录运行**：
   ```bash
   pueue add -w /path/to/project "./script.sh"
   ```

4. **等待任务完成后执行另一个**：
   ```bash
   pueue add -a 1 "npm run build"
   ```

5. **立即开始任务**：
   ```bash
   pueue add -i "npm run dev"
   ```

6. **查看最近20行日志**：
   ```bash
   pueue log -l 20 5
   ```

7. **重启失败的任务**：
   ```bash
   pueue restart --all-failed
   ```

8. **创建并行分组**：
   ```bash
   pueue group add -p 4 compile
   ```

9. **杀死所有运行中的任务**：
   ```bash
   pueue kill -a
   ```

10. **暂停所有，等待当前任务完成**：
    ```bash
    pueue pause -wa
    ```

11. **重启失败的任务**：
    ```bash
    pueue restart --all-failed
    ```

- 确保 `pueued` 守护进程正在运行
- 任务 ID 在重启后会变化（除非使用 `-i, --in-place`）
- 分组可以帮助组织不同类型的任务
- 使用标签可以更容易识别任务
- 使用 `--json` 参数可以方便程序解析输出
- 移除任务前需先终止运行中的任务

## 完整命令列表

| 命令 | 功能 |
|------|------|
| `pueue add` | 添加任务 |
| `pueue status` | 查看状态 |
| `pueue log` | 查看日志 |
| `pueue follow` | 跟随输出 |
| `pueue wait` | 等待完成 |
| `pueue pause` | 暂停任务 |
| `pueue start` | 开始任务 |
| `pueue kill` | 终止任务 |
| `pueue restart` | 重启任务 |
| `pueue stash` | 暂存任务 |
| `pueue enqueue` | 入队任务 |
| `pueue clean` | 清理任务 |
| `pueue reset` | 重置队列 |
| `pueue remove` | 移除任务 |
| `pueue switch` | 交换位置 |
| `pueue send` | 发送输入 |
| `pueue edit` | 编辑任务 |
| `pueue group add` | 添加分组 |
| `pueue group remove` | 删除分组 |
| `pueue parallel` | 设置并行 |

## 延迟格式参考

Pueue 支持灵活的日期时间格式：

```
2020-04-01T18:30:00    # RFC 3339 时间戳
2020-4-1 18:2:30       # 可选前导零
2020-4-1 5:30pm        # 下午5:30
April 1 2020 18:30:00  # 英文月份
1 Apr 8:30pm           # 省略年份
4/1                    # 美国日期格式
wednesday 10:30pm      # 最近周三22:30
wednesday              # 最近周三
4 months               # 4个月后
1 week                 # 1周后
3h                     # 3小时后
3600s                  # 3600秒后
```
