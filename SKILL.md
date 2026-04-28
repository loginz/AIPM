---
name: chat-archiver
description: 自动保存聊天记录和上传的文件/图片到按月份组织的目录结构。图片自动调整尺寸（宽度最大720px，保持比例）并嵌入到Markdown日志中。当用户发送文件或图片，或需要记录聊天历史时使用此技能。
---

# Chat Archiver

聊天记录和文件归档技能。自动组织聊天内容、图片和文件到月度目录结构中，并优化图片尺寸以节省存储空间。

## 目录结构

```
/root/.openclaw/workspace-security/chat-records/
└── 2026-04/
    ├── 2026-04-25-chat-log.md
    ├── 2026-04-25-092100-address.jpg
    └── 2026-04-25-100800-lormelayu.jpg
```

## 文件命名规则

- **月度目录:** `YYYY-MM` 格式
- **聊天日志:** `YYYY-MM-DD-chat-log.md`
- **图片/文件:** `YYYY-MM-DD-HHMMSS-描述.扩展名`

## 工作流程

### 1. 初始化月度目录

```bash
mkdir -p /root/.openclaw/workspace-security/chat-records/$(date +%Y-%m)
```

### 2. 处理图片

使用 `scripts/save_image.py` 脚本自动调整图片尺寸：

```bash
python3 scripts/save_image.py <源路径> <目标路径> [最大宽度]
```

**示例：**
```bash
python3 /root/.openclaw/workspace-security/skills/chat-archiver/scripts/save_image.py \
  /root/.openclaw/media/inbound/image.jpg \
  /root/.openclaw/workspace-security/chat-records/2026-04/2026-04-25-100800-photo.jpg \
  720
```

**输出格式：**
```json
{
  "success": true,
  "source_path": "/path/to/source.jpg",
  "target_path": "/path/to/target.jpg",
  "original_width": 960,
  "original_height": 1280,
  "original_size": "81KB",
  "resized": true,
  "final_width": 720,
  "final_height": 960,
  "final_size": "48KB"
}
```

### 3. 更新聊天日志

在当月当日的 `chat-log.md` 中添加记录：

```markdown
### 2026-04-25 10:08 GMT+8

**用户:** XJ ZHAO (老赵)

**消息内容:** "消息文本"

### 📎 文件接收

- **文件名:** 原始文件名
- **保存路径:** 完整保存路径
- **时间:** 时间戳
- **类型:** 图片/文件类型
- **说明:** 文件说明
- **尺寸调整:** 原始尺寸 → 调整后尺寸（如已调整）
- **文件大小:** 文件大小

![图片描述](./相对路径)
```

## 依赖工具

### 系统工具
- `ls`, `mkdir`, `cp` - 文件管理
- `date` - 时间戳生成

### ImageMagick
- `identify` - 获取图片信息
- `convert` - 图片尺寸调整

**安装 ImageMagick（如未安装）：**
```bash
apt-get update && apt-get install -y imagemagick
```

## 配置参数

- **最大图片宽度:** 720px（默认）
- **工作目录:** `/root/.openclaw/workspace-security/chat-records/`
- **图片格式支持:** JPEG, PNG, GIF, WebP 等（ImageMagick支持的所有格式）

## 使用示例

### 场景1：用户发送图片

1. 检查并创建月度目录
2. 使用 `save_image.py` 处理图片
3. 在聊天日志中记录图片信息
4. 使用相对路径嵌入图片

### 场景2：用户发送文件

1. 检查并创建月度目录
2. 复制文件到月度目录，按时间戳命名
3. 在聊天日志中记录文件信息

### 场景3：纯文本消息

1. 在聊天日志中追加消息记录
2. 包含时间、用户、内容

## 注意事项

- 图片仅当宽度超过最大值时才调整，否则保持原尺寸
- 使用相对路径嵌入图片，便于在不同Markdown查看器中显示
- 每日聊天记录追加到同一个 `chat-log.md` 文件中
- 月度目录自动创建，无需手动维护
