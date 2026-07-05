# 架构设计

## 总体架构

```
┌─────────────────────────────────────────────────┐
│                  Vue 3 Frontend                 │
│  Views → Stores (Pinia) → API (Axios)          │
│  Composables (useCanvasPreview, useFramePlayer) │
└──────────────────┬──────────────────────────────┘
                   │ HTTP/WebSocket
┌──────────────────▼──────────────────────────────┐
│               Go Backend (Gin)                  │
│  Handler → Service → Database (SQLite)         │
└─────────────────────────────────────────────────┘
```

## 后端分层

### Handler 层 (`internal/api/`)
HTTP 请求处理，参数解析，响应格式化。只做薄薄一层。

```go
func (s *Server) listXmlAudio(c *gin.Context) {
    query := c.Query("query")
    records, _, stats, _ := s.db.ListAudioXml(query, "all", 1, -1)
    respondSuccess(c, gin.H{"items": records, "stats": stats})
}
```

### Service 层 (`internal/service/`)
业务逻辑，数据一致性维护，事务管理。

核心服务：
- `NpkService` — NPK 文件加载/解析/保存、Album 管理、Sprite 编辑
- `MusicService` — 音频扫描/索引
- `SoundPackService` — 音效包解密/读写
- `VideoService` — 视频管理

### Database 层 (`internal/database/`)
纯 SQL 操作，不包含业务逻辑。每个领域一个文件。

通用工具：
- `BuildInClause[T]` — 泛型 IN 占位符构建器
- `ensureSpritesLoaded` — 按需从文档缓存或磁盘加载 album sprites

### NPK/IMG 库 (`internal/npk/`)
DNF 二进制格式读写：
- `reader.go` — 通用读取逻辑、颜色格式/压缩模式枚举
- `reader_v2.go` — V2 IMG 格式解析
- `reader_v5.go` — V5 IMG 格式解析（DDS 纹理）
- `writer_v2.go` — V2 IMG 序列化
- `writer_v5.go` — V5 IMG 序列化
- `writer.go` — NPK 写入

## 前端架构

### 路由（懒加载）
只有 Dashboard 首屏加载，其余 16 个页面按需 `() => import(...)` 加载。

### Store 设计（Pinia）
| Store | 职责 |
|-------|------|
| `npk` | NPK 列表、Album 列表、Sprite 列表 |
| `document` | IMG 编辑文档模型、修改追踪、坐标导入导出 |
| `imgEditor` | 帧编辑状态、缓存管理、批量操作 |
| `cache` | 内存/磁盘缓存、LRU 淘汰 |
| `music/video/soundpack` | 媒体资源管理 |
| `theme` | 主题切换 |
| `ui` | UI 状态、头像管理 |

### Composables
- `useCanvasPreview` — 画布渲染 + 去黑底 + 背景模式
- `useFramePlayer` — 帧播放控制（PLAY_INTERVAL = 200ms）
- `useWebSocketPool` — WebSocket 连接池 + 心跳 + 重连

## 关键设计决策

### IMG 编辑 - 双数据源同步
```
s.files[npkName]        ← 初始扫描索引（元数据）
s.documents[npkName]    ← 编辑器加载的数据（完整帧信息）
```
增删改操作同时更新两个数据源，`ensureSpritesLoaded` 按需从文档缓存或磁盘加载。

### Audio XML - 数据库存储
```
audio.xml → ImportAudioXml → SQLite → CRUD → ExportAudioXml → audio.xml
```
SQLite 做数据源，XML 导入导出按需触发。列表走 SQL 索引查询，无需每次全量解析。

### 配置路径自动推导
用户只需设置 NPK 目录（`ImagePacks2`），其他路径自动推导：
```
根目录 = NPK目录去掉末尾 /ImagePacks2
Music     = 根目录/Music
SoundPack = 根目录/SoundPacks
Video     = 根目录/Video
Audio.xml = 根目录/Audio.xml
```
