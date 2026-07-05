# XML 音频编辑器

管理 DNF 的 `audio.xml` 文件，支持 EFFECT/RANDOM/GROUP/MUSIC/AMBIENT 五种条目的增删改查。

## 条目类型

| 类型 | 说明 | 关键字段 |
|------|------|---------|
| EFFECT | 音效定义 | FILE(sounds\xxx.ogg)、音量、优先级、延迟 |
| RANDOM | 随机播放池 | ITEM 列表（TAG + PROB 概率） |
| GROUP | 顺序播放序列 | ITEM 列表（TAG + DELAY 延迟） |
| MUSIC | 音乐定义 | FILE(music\xxx.ogg) |
| AMBIENT | 环境音效 | FILE(sounds\xxx.ogg) |

## 引用关系

```
GROUP → ITEM.tag → RANDOM → ITEM.tag → EFFECT → FILE
                               → MUSIC → FILE
                               → AMBIENT → FILE
```

分组视图自动解析引用链，展开显示完整的引用关系。

## 功能

### 列表浏览
- **分组/平铺切换**：分组视图按 EFFECT/MUSIC/AMBIENT 分组，展开显示引用条目
- **筛选**：按类型筛选（MUSIC/EFFECT/RANDOM/GROUP/AMBIENT）
- **搜索**：按 ID 或 FILE 路径搜索
- **缺失标记**：自动检测音频文件是否存在（磁盘 + 音乐库 + 音效库）

### 编辑
- **添加条目**：选择类型后填写对应字段
  - EFFECT/AMBIENT：FILE 输入带 `sounds\` 前缀，可选取音效包文件
  - MUSIC：FILE 输入带 `music\` 前缀，可选取音乐库文件
  - RANDOM：ITEM 列表，TAG + 概率（必须合计 100%）
  - GROUP：ITEM 列表，TAG + 延迟
- **编辑/删除**：修改已有条目
- **查重**：新增时检查是否已存在相同 type+id

### 播放
- **直接播放**：EFFECT/MUSIC/AMBIENT 播放入口
- **概率播放**：RANDOM 按 ITEM 概率加权随机选择后播放
- **编辑对话框试听**：每个 ITEM 可单独播放预览
- **流式查找**：自动从磁盘/音乐库/音效包 NPK 中查找音频文件

### 文件管理
- **选择 XML**：目录浏览选择 audio.xml 文件
- **导出 XML**：手动触发数据库 → XML 写入
- **检查缺失**：单独触发全量文件存在性检查

## 数据库存储

XML 导入 SQLite，CRUD 操作走数据库，导出按需触发：
```
audio.xml → ImportAudioXml → SQLite → ExportAudioXml → audio.xml
```

## 路径前缀约定

| 类型 | FILE 格式 |
|------|----------|
| MUSIC | `music\filename.ogg` |
| EFFECT | `sounds\path\to\file.ogg` |
| AMBIENT | `sounds\path\to\file.ogg` |
