# NPK Server

DNF 游戏资源管理服务器，支持 NPK 文件管理、IMG 帧编辑、音乐/音效/视频管理、XML 音频编辑。

**技术栈**: Go + Gin + SQLite / Vue 3 + Element Plus + Pinia

## 快速开始

```bash
cd frontend && npm run build
cd .. && cp -r frontend/dist backend/internal/web/dist
cd backend
CGO_ENABLED=0 GOOS=windows GOARCH=amd64 go build -ldflags="-s -w" -o bin/npk-server-embed.exe ./cmd/server
```

## 目录结构

```
npk-server/
├── frontend/           # Vue 3 前端（Element Plus + Pinia）
│   └── src/
│       ├── api/        # API 请求封装
│       ├── stores/     # Pinia 状态管理
│       ├── views/      # 页面组件
│       ├── composables/# 可复用组合函数
│       ├── utils/      # 工具函数
│       └── router/     # 路由配置（懒加载）
├── backend/            # Go 后端（Gin + SQLite）
│   └── internal/
│       ├── api/        # HTTP 处理器 + 路由
│       ├── service/    # 业务逻辑层
│       ├── database/   # 数据访问层
│       ├── config/     # 配置管理
│       ├── npk/        # NPK/IMG 读写库
│       └── video/      # FFmpeg 视频处理
└── docs/               # 文档
```

## 功能模块

| 模块           | 功能                                                     |
| -------------- | -------------------------------------------------------- |
| NPK 管理       | 浏览/搜索/扫描 NPK 文件，查看 Album 和帧信息             |
| IMG 编辑       | 帧属性修改、坐标导入导出、批量替换、引用帧管理、对比模式 |
| 动画播放       | 帧序列播放，支持 V2/V4/V5/V6 格式                        |
| 音乐管理       | 扫描/播放/转换音频（OGG/MP3/FLAC/WAV），FFmpeg 格式转换  |
| 音效管理       | NPK SoundPack 解密、OGG 条目浏览/导入/导出/批量操作      |
| 视频管理       | 视频文件扫描/播放/转换                                   |
| XML 音频编辑器 | audio.xml 解析/编辑，分组引用视图，概率播放              |
| 数据库管理     | SQLite 数据库统计、压缩、重建                            |
| 缓存管理       | LRU 缓存策略，内存/磁盘双层缓存                          |
| 设置           | 目录配置（自动推导）、头像管理、主题切换                 |

## 相关文档

- [架构设计](ARCHITECTURE.md)
- [API 文档](API_DOCS.md)
- [数据库设计](DATABASE_SCHEMA.md)
- [XML 音频编辑器](XML_AUDIO_EDITOR.md)
